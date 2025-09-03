
---

## 🔑 O que é JWT?

O **JWT** é um padrão aberto (RFC 7519) usado para **autenticação e troca de informações** de forma segura entre duas partes. Ele é um **token assinado digitalmente** (normalmente com **HMAC** ou **RSA**) que garante a integridade dos dados.

A estrutura de um JWT é composta por três partes:

```
header.payload.signature
```

- **Header** → Tipo do token e algoritmo de assinatura.
    
- **Payload** → Dados do usuário (claims, como `id`, `email`, `roles`).
    
- **Signature** → Garante que o token não foi alterado.
    

---

## ✅ Pontos Positivos do JWT

- **Sem estado (stateless)**: o servidor não precisa armazenar sessão, pois todas as informações estão no token.
    
- **Escalabilidade**: funciona bem em ambientes distribuídos (ex: microservices).
    
- **Flexível**: pode carregar dados adicionais além da autenticação.
    
- **Compatível**: usado em REST, GraphQL, WebSocket, etc.
    
- **Seguro**: quando bem configurado (expiração curta + refresh token + HTTPS).
    

---

## ❌ Pontos Negativos do JWT

- **Não revogável facilmente**: diferente de sessões, não há como "invalidar" um token antes de expirar sem criar um mecanismo extra (ex: blacklist).
    
- **Tamanho**: o token pode ficar grande, já que carrega dados.
    
- **Persistência insegura**: armazenar JWT no `localStorage` pode abrir portas para ataques XSS.
    
- **Expiração crítica**: se expirar rápido demais, gera problemas de usabilidade; se for longo demais, aumenta riscos de segurança.
    
- **Complexidade**: mais difícil de gerenciar comparado a cookies/sessões tradicionais.
    

---

## ⚙️ Implementando JWT em uma API C# (.NET 6/7+)

### 1. Adicionar os pacotes necessários

No seu projeto API:

```bash
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
```

### 2. Configurar no `Program.cs`

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;
using System.Text;

var builder = WebApplication.CreateBuilder(args);

// Chave secreta (em produção, use Secret Manager ou Azure Key Vault)
var key = Encoding.ASCII.GetBytes("minha-chave-super-secreta");

// Configuração de autenticação com JWT
builder.Services.AddAuthentication(options =>
{
    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
})
.AddJwtBearer(options =>
{
    options.RequireHttpsMetadata = true;
    options.SaveToken = true;
    options.TokenValidationParameters = new TokenValidationParameters
    {
        ValidateIssuer = false, // pode habilitar em produção
        ValidateAudience = false, // idem
        ValidateLifetime = true,
        ValidateIssuerSigningKey = true,
        IssuerSigningKey = new SymmetricSecurityKey(key),
        ClockSkew = TimeSpan.Zero // sem tolerância para expiração
    };
});

builder.Services.AddAuthorization();

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();

app.Run();
```

---

### 3. Gerar o Token

Crie um endpoint, por exemplo em um `AuthController.cs`:

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.IdentityModel.Tokens;
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using System.Text;

[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    [HttpPost("login")]
    public IActionResult Login([FromBody] LoginModel model)
    {
        if (model.Username == "admin" && model.Password == "123") // Exemplo
        {
            var tokenHandler = new JwtSecurityTokenHandler();
            var key = Encoding.ASCII.GetBytes("minha-chave-super-secreta");

            var tokenDescriptor = new SecurityTokenDescriptor
            {
                Subject = new ClaimsIdentity(new[]
                {
                    new Claim(ClaimTypes.Name, model.Username),
                    new Claim(ClaimTypes.Role, "Admin")
                }),
                Expires = DateTime.UtcNow.AddHours(1), // expira em 1h
                SigningCredentials = new SigningCredentials(
                    new SymmetricSecurityKey(key),
                    SecurityAlgorithms.HmacSha256Signature
                )
            };

            var token = tokenHandler.CreateToken(tokenDescriptor);
            var tokenString = tokenHandler.WriteToken(token);

            return Ok(new { token = tokenString });
        }

        return Unauthorized();
    }
}

public class LoginModel
{
    public string Username { get; set; }
    public string Password { get; set; }
}
```

---

### 4. Proteger Endpoints

Em qualquer controller, basta usar `[Authorize]`:

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;

[ApiController]
[Route("api/[controller]")]
public class TesteController : ControllerBase
{
    [HttpGet("publico")]
    public IActionResult Publico() => Ok("Qualquer um acessa.");

    [Authorize]
    [HttpGet("privado")]
    public IActionResult Privado() => Ok("Somente usuários autenticados.");
}
```

---

👉 Resumindo:

- JWT é ótimo para APIs **sem estado** e escaláveis.
    
- Deve ser usado com **expiração curta**, **refresh tokens** e **HTTPS**.
    
- Em C#, basta configurar o `JwtBearer`, gerar tokens e proteger endpoints com `[Authorize]`.
    

---

---

# 🔄 Como funciona o Refresh Token

1. O usuário faz login → recebe **2 tokens**:
    
    - **Access Token** (JWT) → curto prazo (ex: 15 minutos).
        
    - **Refresh Token** (string aleatória) → longo prazo (ex: 7 dias).
        
2. Quando o **access token expira**, o cliente chama a API com o **refresh token** para pedir um novo par de tokens.
    
3. O refresh token é armazenado no **servidor/banco** para poder ser validado e invalidado (diferente do JWT que é stateless).
    

---

# ⚙️ Implementação em C# (.NET 6/7+)

### 1. Modelo para armazenar Refresh Token

```csharp
public class RefreshToken
{
    public string Token { get; set; }
    public string Username { get; set; }
    public DateTime Expiration { get; set; }
    public bool IsRevoked { get; set; }
}
```

No exemplo simples, vamos armazenar em **memória**. Em produção, use **banco de dados**.

```csharp
public static class RefreshTokenStore
{
    public static List<RefreshToken> Tokens = new();
}
```

---

### 2. Gerar Access + Refresh Token no Login

No `AuthController.cs`:

```csharp
[HttpPost("login")]
public IActionResult Login([FromBody] LoginModel model)
{
    if (model.Username == "admin" && model.Password == "123") // Exemplo
    {
        var accessToken = GenerateJwtToken(model.Username);
        var refreshToken = GenerateRefreshToken(model.Username);

        return Ok(new
        {
            accessToken,
            refreshToken
        });
    }

    return Unauthorized();
}

private string GenerateJwtToken(string username)
{
    var tokenHandler = new JwtSecurityTokenHandler();
    var key = Encoding.ASCII.GetBytes("minha-chave-super-secreta");

    var tokenDescriptor = new SecurityTokenDescriptor
    {
        Subject = new ClaimsIdentity(new[]
        {
            new Claim(ClaimTypes.Name, username),
            new Claim(ClaimTypes.Role, "Admin")
        }),
        Expires = DateTime.UtcNow.AddMinutes(15), // Expira rápido
        SigningCredentials = new SigningCredentials(
            new SymmetricSecurityKey(key),
            SecurityAlgorithms.HmacSha256Signature
        )
    };

    var token = tokenHandler.CreateToken(tokenDescriptor);
    return tokenHandler.WriteToken(token);
}

private string GenerateRefreshToken(string username)
{
    var refreshToken = new RefreshToken
    {
        Token = Convert.ToBase64String(Guid.NewGuid().ToByteArray()),
        Username = username,
        Expiration = DateTime.UtcNow.AddDays(7), // 7 dias
        IsRevoked = false
    };

    RefreshTokenStore.Tokens.Add(refreshToken);

    return refreshToken.Token;
}
```

---

### 3. Endpoint para renovar o Access Token

```csharp
[HttpPost("refresh")]
public IActionResult Refresh([FromBody] string refreshToken)
{
    var storedToken = RefreshTokenStore.Tokens
        .FirstOrDefault(t => t.Token == refreshToken && !t.IsRevoked);

    if (storedToken == null || storedToken.Expiration < DateTime.UtcNow)
        return Unauthorized("Refresh token inválido ou expirado.");

    // Revogar token antigo (opcional, dependendo da sua estratégia)
    storedToken.IsRevoked = true;

    // Gerar novos tokens
    var newAccessToken = GenerateJwtToken(storedToken.Username);
    var newRefreshToken = GenerateRefreshToken(storedToken.Username);

    return Ok(new
    {
        accessToken = newAccessToken,
        refreshToken = newRefreshToken
    });
}
```

---

# 🚀 Fluxo resumido

1. **Login** → cliente recebe `{ accessToken, refreshToken }`.
    
2. Usa o **access token** para chamar a API (até expirar).
    
3. Quando expira → chama `/refresh` com o **refresh token**.
    
4. Recebe **novo par** `{ accessToken, refreshToken }`.
    
5. O refresh antigo pode ser **revogado** para maior segurança.
    

---

👉 Assim você mantém **tokens curtos (seguros)** e um mecanismo de renovação controlado.

Perfeito 🙌 Vamos evoluir o exemplo para usar **Entity Framework Core** e persistir os refresh tokens em banco de dados. Assim você terá um fluxo **mais próximo de produção**.

---
---

# 🏗️ Implementação com Entity Framework

### 1. Criar o modelo do RefreshToken

```csharp
public class RefreshToken
{
    public int Id { get; set; }
    public string Token { get; set; }
    public string Username { get; set; }
    public DateTime Expiration { get; set; }
    public bool IsRevoked { get; set; }
}
```

---

### 2. Criar o `DbContext`

```csharp
using Microsoft.EntityFrameworkCore;

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    public DbSet<RefreshToken> RefreshTokens { get; set; }
}
```

No `Program.cs`, registrar o contexto:

```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseInMemoryDatabase("AuthDb")); // Para testes, banco em memória
```

_(em produção, use `UseSqlServer` ou outro provedor)._

---

### 3. Atualizar `AuthController` para usar banco

```csharp
[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    private readonly AppDbContext _context;

    public AuthController(AppDbContext context)
    {
        _context = context;
    }

    [HttpPost("login")]
    public IActionResult Login([FromBody] LoginModel model)
    {
        if (model.Username == "admin" && model.Password == "123") // Exemplo
        {
            var accessToken = GenerateJwtToken(model.Username);
            var refreshToken = GenerateRefreshToken(model.Username);

            return Ok(new
            {
                accessToken,
                refreshToken
            });
        }

        return Unauthorized();
    }

    private string GenerateJwtToken(string username)
    {
        var tokenHandler = new JwtSecurityTokenHandler();
        var key = Encoding.ASCII.GetBytes("minha-chave-super-secreta");

        var tokenDescriptor = new SecurityTokenDescriptor
        {
            Subject = new ClaimsIdentity(new[]
            {
                new Claim(ClaimTypes.Name, username),
                new Claim(ClaimTypes.Role, "Admin")
            }),
            Expires = DateTime.UtcNow.AddMinutes(15),
            SigningCredentials = new SigningCredentials(
                new SymmetricSecurityKey(key),
                SecurityAlgorithms.HmacSha256Signature
            )
        };

        var token = tokenHandler.CreateToken(tokenDescriptor);
        return tokenHandler.WriteToken(token);
    }

    private string GenerateRefreshToken(string username)
    {
        var refreshToken = new RefreshToken
        {
            Token = Convert.ToBase64String(Guid.NewGuid().ToByteArray()),
            Username = username,
            Expiration = DateTime.UtcNow.AddDays(7),
            IsRevoked = false
        };

        _context.RefreshTokens.Add(refreshToken);
        _context.SaveChanges();

        return refreshToken.Token;
    }

    [HttpPost("refresh")]
    public IActionResult Refresh([FromBody] string refreshToken)
    {
        var storedToken = _context.RefreshTokens
            .FirstOrDefault(t => t.Token == refreshToken && !t.IsRevoked);

        if (storedToken == null || storedToken.Expiration < DateTime.UtcNow)
            return Unauthorized("Refresh token inválido ou expirado.");

        // Revogar token antigo
        storedToken.IsRevoked = true;
        _context.RefreshTokens.Update(storedToken);
        _context.SaveChanges();

        // Gerar novos tokens
        var newAccessToken = GenerateJwtToken(storedToken.Username);
        var newRefreshToken = GenerateRefreshToken(storedToken.Username);

        return Ok(new
        {
            accessToken = newAccessToken,
            refreshToken = newRefreshToken
        });
    }
}

public class LoginModel
{
    public string Username { get; set; }
    public string Password { get; set; }
}
```

---

# 🔐 Boas práticas em produção

- Usar **chave secreta forte** e armazená-la em **Azure Key Vault, AWS Secrets Manager** ou similar.
    
- Armazenar os **refresh tokens no banco** e associá-los a um **usuário** real.
    
- Permitir múltiplos refresh tokens por usuário (para suportar login em dispositivos diferentes).
    
- Usar **JWT Access Tokens curtos (5–15 minutos)**.
    
- Revogar tokens antigos quando o usuário fizer logout.
    
- Usar **HTTPS sempre** (nunca trafegar tokens em HTTP).
    

---

👉 Agora você tem um fluxo completo:

- **Login** → gera JWT (curto) + Refresh Token (longo, salvo no banco).
    
- **Refresh** → renova os tokens e revoga o antigo.
    

---
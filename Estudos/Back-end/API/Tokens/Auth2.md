
---

## 🔐 2. OAuth2

O **OAuth2** é um **protocolo de autorização** amplamente usado (Google, GitHub, Facebook, etc.). Ele permite que aplicações acessem recursos protegidos em nome do usuário sem expor diretamente a senha.

### Fluxos mais comuns:

- **Authorization Code** (para apps web)
    
- **Client Credentials** (para comunicação servidor-servidor)
    
- **Password Credentials** (menos usado hoje em dia)
    

👉 Exemplo mais comum em APIs: **Client Credentials**

1. O cliente envia `client_id` e `client_secret` para o servidor de autenticação.
    
2. O servidor retorna um **access_token** (que geralmente é um JWT).
    
3. Esse `access_token` é usado nas requisições como um Bearer Token.
    

### Postman:

- Aba **Authorization** → Selecione **OAuth 2.0**.
    
- Clique em **Get New Access Token** e configure:
    
    - **Auth URL**: URL de autorização do servidor OAuth.
        
    - **Access Token URL**: URL para obter o token.
        
    - **Client ID** e **Client Secret**: credenciais fornecidas.
        
    - **Scope**: permissões solicitadas.
        
- Após obter, clique em **Use Token**.
    
- O Postman adiciona o token como Bearer:
    

```postman
 Authorization: Bearer <access_token>
```

---

## 🔐 1. Pontos Positivos do OAuth2

- **Segurança elevada** → Não expõe diretamente as credenciais do usuário (username/senha), apenas tokens.
    
- **Granularidade de permissões** → Você pode controlar escopos (ex: `read:users`, `write:products`).
    
- **Padronização** → É um protocolo bem estabelecido, usado por Google, GitHub, Facebook etc.
    
- **Suporte a múltiplos fluxos** → Pode ser usado em **aplicações web**, **mobile**, **APIs servidor-servidor**.
    
- **Integração com terceiros** → Ideal para login social e delegação de acesso.
    

---

## ⚠️ 2. Pontos Negativos do OAuth2

- **Complexidade** → Implementar corretamente é mais difícil do que um simples JWT ou API Key.
    
- **Manutenção** → É necessário gerenciar tokens de acesso e refresh, além de configurar corretamente os escopos.
    
- **Sobrecarga** → Em cenários simples, pode ser "exagerado" usar OAuth2.
    
- **Dependência de configuração** → Se for usar com provedores externos (Google, Facebook, etc.), depende da política e disponibilidade deles.
    

---

## ⚙️ 3. Implementando OAuth2 em uma API C# (ASP.NET Core)

### 🔹 Passo 1 – Criar projeto API

```bash
dotnet new webapi -n MinhaApiOAuth
cd MinhaApiOAuth
```

---

### 🔹 Passo 2 – Configurar autenticação com **OAuth2 + JWT Bearer**

No `Program.cs` (ou `Startup.cs` em versões antigas), adicione:

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;
using System.Text;

var builder = WebApplication.CreateBuilder(args);

// Configuração do OAuth2/JWT
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = "https://meu-servidor-oauth.com",
            ValidAudience = "https://minhaapi.com",
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes("minha_chave_super_secreta"))
        };
    });

builder.Services.AddAuthorization();

builder.Services.AddControllers();

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();

app.Run();
```

---

### 🔹 Passo 3 – Proteger um endpoint

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;

namespace MinhaApiOAuth.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class ProdutosController : ControllerBase
    {
        [HttpGet]
        [Authorize] // exige token válido
        public IActionResult GetProdutos()
        {
            var produtos = new[] {
                new { Id = 1, Nome = "Notebook" },
                new { Id = 2, Nome = "Smartphone" }
            };
            return Ok(produtos);
        }

        [HttpGet("publico")]
        public IActionResult GetPublico()
        {
            return Ok("Esse endpoint é público, não precisa de token.");
        }
    }
}
```

---

### 🔹 Passo 4 – Gerar tokens (Servidor OAuth)

Aqui você tem 2 opções:

1. Usar um **servidor de identidade pronto** → Ex: **IdentityServer4**, **Duende IdentityServer**, ou serviços externos (Google, Auth0, Azure AD).
    
2. Implementar um **controller de login** que retorna um JWT (simulando o fluxo Password Credentials).
    

Exemplo simples (não recomendado em produção, apenas para estudo):

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.IdentityModel.Tokens;
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using System.Text;

namespace MinhaApiOAuth.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class AuthController : ControllerBase
    {
        [HttpPost("login")]
        public IActionResult Login([FromBody] LoginRequest request)
        {
            if (request.Username == "admin" && request.Password == "123")
            {
                var claims = new[]
                {
                    new Claim(ClaimTypes.Name, request.Username),
                    new Claim("scope", "read:produtos")
                };

                var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes("minha_chave_super_secreta"));
                var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

                var token = new JwtSecurityToken(
                    issuer: "https://meu-servidor-oauth.com",
                    audience: "https://minhaapi.com",
                    claims: claims,
                    expires: DateTime.Now.AddHours(1),
                    signingCredentials: creds);

                return Ok(new { access_token = new JwtSecurityTokenHandler().WriteToken(token) });
            }

            return Unauthorized();
        }
    }

    public class LoginRequest
    {
        public string Username { get; set; }
        public string Password { get; set; }
    }
}
```

---

### 🔹 Passo 5 – Testando no **Postman**

1. Fazer login:

- POST → `https://localhost:5001/api/auth/login`

- Body (JSON):

  ```json
	{
    "username": "admin",
    "password": "123"
	}
```

- Resposta:

```json
 {
	 "access_token": "eyJhbGciOiJIUzI1NiIsInR..."
 }
 ```

1. Usar o token no endpoint protegido:

- GET → `https://localhost:5001/api/produtos`

- Headers → `Authorization: Bearer <access_token>`

- Resposta:

```json	
[
	  { "id": 1, "nome": "Notebook" },
	  { "id": 2, "nome": "Smartphone" }
]
```


---

👉 Esse exemplo é um **OAuth2 simplificado** (fluxo de senha + geração de JWT).  
Em produção, normalmente usamos **IdentityServer4/Duende** ou **serviços externos (Auth0, Azure AD, Google Identity)** para gerenciar **refresh tokens**, **escopos**, **usuários**, etc.

---
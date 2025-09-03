
---

## 🔑 1. Definição

A **API Key** é um **identificador único** (geralmente uma string longa e aleatória) usada para autenticar clientes que consomem uma API.  
Em vez de representar um **usuário específico** (como no OAuth2 ou JWT), a chave identifica **a aplicação** ou **o cliente** que está acessando.

---

## ✅ 2. Pontos Positivos

- **Simplicidade** → Fácil de gerar, distribuir e validar.
    
- **Baixa complexidade** → Sem necessidade de protocolos avançados (como OAuth2).
    
- **Bom para uso interno** → APIs internas ou microserviços podem usá-la de forma rápida.
    
- **Controle básico de acesso** → É possível habilitar/desabilitar rapidamente chaves de clientes.
    

---

## ⚠️  3. Pontos Negativos

- **Menos seguro** → Se a chave for exposta, qualquer pessoa pode usá-la.
    
- **Não identifica usuários** → Só autentica a aplicação, não quem está por trás dela.
    
- **Sem granularidade** → Geralmente não há escopos ou permissões detalhadas.
    
- **Difícil de revogar isoladamente** → Normalmente precisa gerar nova chave para substituir.

---

## 🧪 4. Testando no Postman

Existem duas formas comuns:

- Vá em **Authorization** → escolha **API Key**.

- Configure:

- **Key** = `x-api-key` (ou o cabeçalho esperado pela API).

- **Value** = `minha_chave_super_secreta`.

- **Add to** → Header.

- O Postman enviará:

### 🔹 Usando no **Header**

```
GET /api/produtos HTTP/1.1
Host: localhost:5001
x-api-key: minha_chave_super_secreta
```

🔹 Usando na **Query String**

```
https://localhost:5001/api/produtos?api_key=minha_chave_super_secreta
```

## ⚙️ 5. Implementando API Key em uma API C#

### 🔹 Passo 1 – Middleware para validar API Key

No `Program.cs` (ou `Startup.cs`), você pode criar um **middleware customizado**:

```c#
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

const string API_KEY_HEADER = "x-api-key";
const string MINHA_CHAVE = "minha_chave_super_secreta";

// Middleware de validação da API Key
app.Use(async (context, next) =>
{
    if (!context.Request.Headers.TryGetValue(API_KEY_HEADER, out var extractedApiKey))
    {
        context.Response.StatusCode = 401;
        await context.Response.WriteAsync("API Key não fornecida.");
        return;
    }

    if (!MINHA_CHAVE.Equals(extractedApiKey))
    {
        context.Response.StatusCode = 403;
        await context.Response.WriteAsync("API Key inválida.");
        return;
    }

    await next();
});

app.MapGet("/api/produtos", () =>
{
    return new[]
    {
        new { Id = 1, Nome = "Notebook" },
        new { Id = 2, Nome = "Smartphone" }
    };
});

app.Run();

```

🔹 Passo 2 – Usando via Query String

Se quiser aceitar via query string também:

```c#
app.Use(async (context, next) =>
{
    string apiKey = context.Request.Headers[API_KEY_HEADER].FirstOrDefault()
                    ?? context.Request.Query["api_key"].FirstOrDefault();

    if (string.IsNullOrEmpty(apiKey))
    {
        context.Response.StatusCode = 401;
        await context.Response.WriteAsync("API Key não fornecida.");
        return;
    }

    if (apiKey != MINHA_CHAVE)
    {
        context.Response.StatusCode = 403;
        await context.Response.WriteAsync("API Key inválida.");
        return;
    }

    await next();
});
```

### 🔹 Passo 3 – Testando

1. Request sem API Key

```
GET /api/produtos
→ Resposta: 401 API Key não fornecida.
```

1. Request com API Key inválida

```
x-api-key: chave_errada
→ Resposta: 403 API Key inválida.
```

2. Request com API Key válida

```
x-api-key: minha_chave_super_secreta
→ Resposta:
[
  { "id": 1, "nome": "Notebook" },
  { "id": 2, "nome": "Smartphone" }
]

```

## 🔍 Resumindo

- **API Key** é simples e rápida, mas não tão segura quanto OAuth2 ou JWT.

- Melhor usar em **APIs internas**, **protótipos**, ou **integrações simples**.

- Implementação em C# pode ser feita via **middleware customizado**.

- No **Postman**, o teste pode ser feito tanto via **Header** quanto via **Query Params**.
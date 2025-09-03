## 🧭 **Passo a Passo para Análise de Software Herdado Incompleto**

### **1. Entenda o contexto do projeto**

Antes de olhar o código, tente levantar o **propósito do sistema**:

- Qual o objetivo do software?
    
- Quem são os usuários finais?
    
- Quais funcionalidades já existem e quais estão faltando?
    
- Existe documentação, backlog, protótipos ou algum levantamento de requisitos?
    

✅ **Ação:** Converse com seu chefe, equipe de produto ou cliente. Documente o que descobrir.

---

### **2. Faça um levantamento do que você recebeu**

Verifique tudo que está incluso no código-fonte:

- Quais pastas e arquivos existem?
    
- Tem algum README, documentação técnica, diagramas ou banco de dados?
    
- Tem dependências externas (bibliotecas, serviços)?
    
- Linguagem e framework usados?
    

✅ **Ação:** Crie uma **estrutura resumida do projeto**, tipo:

`📁 src   ┣ 📁 controllers   ┣ 📁 models   ┣ 📁 views 📄 package.json 📄 README.md`

---

### **3. Rode o projeto localmente (se possível)**

Antes de analisar código, veja o sistema funcionando:

- Tente instalar dependências e rodar local.
    
- Veja se há mensagens de erro.
    
- Use ferramentas como Docker, caso seja necessário.
    

✅ **Ação:** Documente o que funcionou e o que não funcionou. Tire prints ou faça anotações.

---

### **4. Mapeie o que está pronto e o que está faltando**

Você precisa identificar:

- Quais funcionalidades estão implementadas?
    
- Existem "TODOs", comentários, códigos comentados?
    
- Qual o fluxo principal do sistema?
    
- Há testes automatizados?
    

✅ **Ferramentas úteis:**

- Ferramentas de inspeção de rotas (ex: Postman, Insomnia)
    
- Code linters e analisadores estáticos (ESLint, SonarLint, etc)
    

---

### **5. Crie um diagrama/resumo da arquitetura**

Não precisa ser complexo:

- Liste os módulos, pacotes ou camadas do sistema.
    
- Mostre como eles se conectam (ex: controller → service → model).
    
- Indique onde há falhas, partes faltando ou mal implementadas.
    

✅ **Ação:** Use ferramentas como Draw.io, Lucidchart ou até papel.

---

### **6. Converse com quem escreveu ou acompanhou o projeto**

Se possível:

- Converse com os desenvolvedores antigos.
    
- Veja se há histórico em Git, issues no Jira, Trello ou similar.
    
- Veja os commits antigos para entender o progresso.
    

---

### **7. Escreva um relatório ou resumo**

Seu chefe vai querer saber:

- O que já funciona?
    
- O que está faltando?
    
- É possível aproveitar esse código?
    
- Que riscos ou problemas você identificou?
    
- Qual a sua sugestão para seguir (reaproveitar, reescrever, refatorar)?
    

✅ **Ação:** Monte um documento com seções como:

- Visão geral
    
- Funcionalidades existentes
    
- Pontos críticos / problemas encontrados
    
- Requisitos pendentes
    
- Recomendação
    

---

## ⚠️ Dicas Finais

- **Não tenha medo de perguntar:** Você é júnior, mas está fazendo um trabalho de nível pleno. Pergunte ao time.
    
- **Faça commits frequentes:** Sempre que entender ou mudar algo, registre no Git.
    
- **Não tente "consertar tudo de uma vez":** Entenda primeiro, depois planeje as mudanças.
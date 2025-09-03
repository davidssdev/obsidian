No Git, convencionou-se o uso de **conventional commits** para manter um histórico de commits mais organizado e semântico. Os principais tipos de commits usados são:

### 1. **feat**

- Indica a adição de uma **nova funcionalidade** ao projeto.
- Exemplo:

```sh
git commit -m "feat: adicionar botão de login"
```
### 2. **fix**

- Usado para corrigir **bugs** no código.
- Exemplo:

```sh
git commit -m "fix: corrigir erro de validação no formulário"
```
### 3. **chore**

- Refere-se a tarefas de manutenção ou mudanças que **não afetam a lógica do código** (como atualizações de dependências, configuração de ferramentas, etc.).
- Exemplo:

```sh
git commit -m "chore: atualizar dependências do projeto"
```


### 4. **docs**

- Usado para alterações na **documentação** (README, comentários, etc.).
- Exemplo:


```sh
git commit -m "docs: adicionar documentação da API"
```


### 5. **style**

- Refere-se a alterações de **formatação**, como espaçamento, indentação, mas que **não alteram a funcionalidade**.
- Exemplo:


```sh
git commit -m "style: ajustar identação do código"
```


### 6. **refactor**

- Indica uma **refatoração** do código, ou seja, melhorias na estrutura sem mudar o comportamento.
- Exemplo:

```sh
git commit -m "refactor: melhorar função de cálculo"
```


### 7. **perf**

- Usado para mudanças que **melhoram o desempenho** do código.
- Exemplo:

```sh
git commit -m "perf: otimizar consulta ao banco de dados"
```


### 8. **test**

- Usado para adicionar ou modificar **testes**.
- Exemplo:


```sh
git commit -m "test: adicionar testes unitários para função de login"
```


### 9. **build**

- Refere-se a alterações que afetam o **sistema de build** ou **ferramentas externas**.
- Exemplo:

```sh
git commit -m "build: configurar pipeline de CI/CD"
```


### 10. **ci**

- Indica mudanças relacionadas a **integração contínua** (CI), como arquivos de configuração de CI/CD.
- Exemplo:

```sh
git commit -m "ci: ajustar workflow do GitHub Actions"
```


Esses tipos de commits ajudam a entender rapidamente a natureza das alterações no código e facilitam a geração automática de changelogs.


**Comando para alteração do e-mail**

### **1. Alterar o e-mail global**

Este comando muda o e-mail usado em todos os repositórios no sistema.

```sh
git config --global user.email "novo-email@example.com"
```

---

### **2. Alterar o e-mail para um repositório específico**

Se você deseja mudar o e-mail apenas para o repositório atual:

```sh
git config user.email "novo-email@example.com"
```

---

### **3. Verificar o e-mail configurado**

- Para verificar a configuração global:
    
    ```sh
    git config --global user.email
```
    
- Para verificar a configuração local (repositório atual):
    
    ```sh
    git config user.email
```
    

---

### **4. Alterar o e-mail em commits antigos**

Se você precisa corrigir o e-mail em commits já realizados, use o comando abaixo com cuidado. Isso reescreve o histórico de commits.

bash

```sh
git filter-repo --email-callback '   if email == "email-antigo@example.com":       return "novo-email@example.com"   return email '
```

**Nota:** Você precisará instalar a ferramenta `git-filter-repo` se ela não estiver disponível no seu sistema. Substitua os e-mails conforme necessário.

---

### **5. Corrigir o autor de um único commit**

Se você deseja corrigir um commit específico que ainda não foi enviado (não está no repositório remoto):

bash

```sh
git commit --amend --author="Nome <novo-email@example.com>"

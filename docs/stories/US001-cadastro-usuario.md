---
StoryVersion: v1
USID: US001
Fonte: Solicitação do time de produto
---

# USER STORY: Cadastro de Usuário

Breve descrição: Permitir que novos usuários se registrem na aplicação com e-mail e senha para obter acesso básico.

## US001: Cadastro de Usuário
**Como** usuário não autenticado,
**Quero** me registrar informando e-mail, senha e confirmação de senha,
**Para** poder acessar a aplicação como usuário registrado.

---

### Critérios de Aceitação
- [ ] Critério 1: Dado que eu não estou autenticado, quando eu acessar a página "Registrar" e preencher um e-mail válido, senha (8-25 caracteres) e confirmação de senha idêntica, então ao clicar em "Cadastrar" o sistema deve persistir o usuário no banco de dados e exibir a mensagem "Registro realizado com sucesso".
- [ ] Critério 2: Dado um e-mail já existente no sistema, quando eu tentar registrar com esse e-mail, então devo ver uma mensagem de erro informando que o e-mail já está em uso.
- [ ] Critério 3: Dado uma senha com menos de 8 ou mais de 25 caracteres, quando eu tentar cadastrar, então devo ver uma validação de erro indicando o comprimento inválido da senha.
- [ ] Critério 4: Dado que a confirmação de senha não coincide com a senha, quando eu tentar cadastrar, então devo ver uma validação de erro indicando que as senhas não coincidem.

---

### Cenários Negativos
- Dado um e-mail com formato inválido, quando eu enviar o formulário, então o registro deve ser recusado e exibir mensagem de erro de formato de e-mail.
- Dado um e-mail duplicado no banco, quando eu enviar o formulário, então o registro deve ser recusado e exibir mensagem "E-mail já cadastrado".

---

### Regras de Negócio
- A senha deve ter entre 8 e 25 caracteres.
- O e-mail deve ser válido conforme padrão de e-mail (ex.: conter `@` e domínio) e único no sistema.
- A confirmação de senha deve ser idêntica ao campo senha.

---

### Exceções / Restrições
- Não inclui verificação ou confirmação por e-mail.
- Não inclui fluxo de recuperação de senha.
- O cadastro é básico; não contempla perfis, papéis ou dados adicionais além dos campos especificados.

---

### Dependências
- Integração com camada de persistência (banco de dados) para salvar o usuário.
- Endpoint/API backend responsável por criar usuários (quando houver separação frontend/backend).

---

### Glossário Local
| Termo | Definição |
|-------|-----------|
| Usuário | Pessoa que se registra e passa a ter credenciais no sistema |

---

### Observações
- Entregas esperadas: página de registro funcional, sistema de validações, integração com banco de dados e mensagens de feedback.
- Sugestão (fora do escopo atual): implementar confirmação por e-mail e política de senha mais robusta (ex.: obrigar caracteres especiais) em iteração futura.
- Próximo artefato esperado: PRD para detalhamento técnico (se desejado) e/ou implementação backend/frontend.

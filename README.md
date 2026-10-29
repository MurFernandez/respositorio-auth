# respositorio-auth

# Introdução

Este repositório contém a documentação e as evidências práticas da atividade de testes de software focada na validação de APIs de autenticação. A prática foi desenvolvida aplicando conceitos de Qualidade de Software no contexto do curso de Análise e Desenvolvimento de Sistemas, utilizando ferramentas modernas de mercado para simular o ambiente de um cliente real consumindo um serviço de backend.

# Objetivo da Atividade

O objetivo principal desta atividade é compreender e aplicar testes de caixa cinza em APIs de autenticação. Para isso, foi exigida a configuração de um ambiente de backend utilizando o Supabase, a criação de utilizadores de teste e a execução de múltiplos cenários de validação através do Postman Desktop. A abordagem de caixa cinza permite testar a aplicação com conhecimento parcial da sua estrutura (endpoints, headers e estrutura JSON), avaliando as respostas HTTP e o comportamento de segurança do servidor sem acesso direto ao seu código-fonte.

# Configuração do Supabase

Para a realização dos testes, o ambiente de backend foi aprovisionado da seguinte forma:
- **Criação do Projeto:** Foi criado um novo projeto gratuito chamado `auth-caixa-cinza` na plataforma Supabase.
- **Tipo de Autenticação:** Foi utilizado o *Email Provider* (Autenticação por e-mail e palavra-passe). Para facilitar os testes automatizados, a opção de confirmação obrigatória de e-mail foi desativada, permitindo a criação de contas já validadas.
- **Criação de Utilizadores:** Através do painel `Authentication > Users`, foi criado o utilizador de teste `people02@email.com`.
- **Objetivo da Configuração:** O objetivo foi gerar uma API real e funcional para expor o endpoint de autenticação. A partir do painel `Project Settings > API`, foram extraídas a *Project URL* e a *API Key (anon/public)*, credenciais essenciais para autorizar as requisições que seriam feitas pelo Postman.

*(Evidências: `01_supabase_criando_usuario.png`, `02_supabase_painel_usuarios.png` na pasta `/prints/`)*

# Configuração do Postman

O Postman Desktop foi configurado para isolar o ambiente de testes e dinamizar as requisições:
- **Criação do Workspace:** Foi criado um Workspace pessoal exclusivo para organizar a *Collection* de testes desta atividade.
- **Configuração do Environment:** Foi criado um ambiente chamado `Supabase - Auth` para armazenar variáveis globais.
- **Finalidade das Variáveis:**
  - `base_url`: Armazena a URL raiz da API do Supabase, facilitando a alteração de ambientes sem reescrever as rotas.
  - `api_key`: Guarda o token público de comunicação com o Supabase.
  - `email`: E-mail do utilizador de teste cadastrado.
  - `password`: Palavra-passe do utilizador de teste.
- **Utilização:** O Postman atua como o cliente (Client) que injeta as cargas de dados (Payloads) na API do Supabase, permitindo analisar os cabeçalhos de resposta, o *Status Code* e o tempo de requisição.

*(Evidências: `03_postman_workspace.png`, `04_postman_environment.png` na pasta `/prints/`)*

# Configuração das Requisições

A comunicação com o Supabase foi feita através de uma requisição estruturada da seguinte forma:
- **Endpoint utilizado:** `/auth/v1/token?grant_type=password`
- **Método HTTP:** `POST`
- **Finalidade:** Enviar as credenciais de acesso em formato JSON para o servidor. Se válidas, a API devolve um *Access Token* JWT.
- **Headers utilizados:** Foram passados o `apikey` e o `Authorization` para identificar a origem permitida da requisição, e o `Content-Type` para informar o servidor de que o corpo da mensagem está em JSON.

### Exemplo de Header

| Header | Valor |
| :--- | :--- |
| apikey | `{{api_key}}` |
| Authorization | Bearer `{{api_key}}` |
| Content-Type | application/json |

### Exemplo do Body JSON

```json``
{
  "email": "people02@email.com",
  "password": "senha_do_usuario"
}
``

# Execução dos Testes
Foram executados obrigatoriamente cinco cenários de testes de caixa cinza (um de sucesso e quatro forçando falhas e exceções).

Cenários executados: Login válido, Senha incorreta, Utilizador inexistente, Campos vazios e Credenciais inválidas (erro de formato).

Resultados e Erros encontrados: O Supabase bloqueou corretamente todas as anomalias, retornando Status 400 Bad Request com a mensagem invalid_credentials ou validation_failed para as falhas, e Status 200 OK apenas para a requisição perfeita.

(Evidências dos 5 cenários disponíveis na pasta /prints/)

# Registro dos Testes
Como os testes foram registados: Os cenários executados foram formalmente documentados numa folha de cálculo (planilha), categorizando o ID do teste, cenário, entradas, resultados previstos, resultados obtidos e o status de conformidade.

Importância da documentação: Registar os testes garante a rastreabilidade do projeto, permitindo comprovar a cobertura de validação e oferecendo um histórico confiável para futuras regressões (Regression Testing).

Falhas identificadas: Não foram identificadas vulnerabilidades na API; o Supabase processou corretamente todas as validações de segurança impostas pelos testes.

(A folha de cálculo encontra-se na diretoria /planilha/ e o print em 11_planilha_registro_testes.png)

# Resultados Obtidos
Os testes demonstraram a robustez da API de autenticação padrão do Supabase:

O servidor garantiu a geração segura de tokens JWT apenas no cenário estrito de credenciais válidas (Status 200).

Requisições malformadas ou com dados ilícitos foram rejeitadas prontamente na camada de validação, retornando Status 400 sem causar quedas no serviço (Status 500) ou fuga de dados.

A API não distingue o erro de "Utilizador inexistente" do erro de "Palavra-passe incorreta" na sua mensagem de resposta, o que é uma excelente prática de segurança para evitar enumeração de utilizadores por parte de atacantes.

# Conclusão
Execução e Funcionamento: A autenticação funcionou plenamente. O planeamento das variáveis no Postman permitiu uma execução dinâmica e limpa.

Dificuldades Encontradas: A principal dificuldade deu-se na fase de configuração inicial das variáveis do Postman, onde o caminho /rest/v1/ incluído acidentalmente na URL base gerou erros de 404 Not Found, sendo resolvido com o ajuste para o endpoint correto de auth. Além disso, foi necessário ajustar a configuração de confirmação de e-mail no painel do Supabase.

Falhas e Melhorias: A API nativa mostrou-se fiável. Uma melhoria futura para a implementação deste backend num sistema de produção seria a ativação de Rate Limiting (para evitar ataques de força bruta iterativos) e a reativação da confirmação obrigatória de e-mail por segurança lógica.

Importância dos Testes Caixa Cinza: Esta metodologia é indispensável. Ao não olhar diretamente para o código-fonte, o Quality Assurance (QA) simula o comportamento real de um cliente HTTP ou de um utilizador mal-intencionado. Validar o contrato da API através das suas entradas e saídas garante que a aplicação não só faz o que é suposto fazer, como também suporta o que não é suposto receber.

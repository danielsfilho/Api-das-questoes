# APIdasQuestões

**Um acervo de questões de concursos públicos, pronto para consumir por API.**

Quem constrói uma plataforma de preparação para concursos descobre cedo que o código é a parte fácil: o difícil é ter questões. Montar um banco próprio significa coletar provas, digitar enunciados, conferir gabaritos, classificar por matéria, banca, cargo e assunto, e manter tudo isso atualizado quando questões são anuladas ou a lei muda.

A APIdasQuestões existe para tirar esse peso de quem está desenvolvendo. Você chama um endpoint REST e recebe questões já classificadas, com enunciado, alternativas e gabarito, prontas para renderizar na sua tela.

### Para quem é

- **Plataformas e cursinhos de preparação para concursos** que querem oferecer resolução de questões, simulados ou listas de exercícios sem manter um banco próprio.
- **Devs que estão tirando uma ideia do papel**: um app de estudo, um bot de questões diárias, um gerador de simulados, um projeto de faculdade ou um MVP para validar com alunos reais.
- **Quem trabalha com no-code** (Lovable, n8n e afins) e precisa de uma fonte de questões confiável para plugar no fluxo.

---

## Links

| | |
|---|---|
| 🌐 Site | https://www.apidasquestoes.com.br |
| 📖 Documentação | https://www.apidasquestoes.com.br/documentacao |
| 🧩 Guias no-code | https://www.apidasquestoes.com.br/guias/no-code |
| 💳 Planos e preços | https://www.apidasquestoes.com.br/planos |
| 📝 Criar conta | https://www.apidasquestoes.com.br/auth/signup |
| 🖥️ Repositório do front-end | https://github.com/danielsfilho/api-de-questoes-front-end |

URL base da API: `https://api.apidasquestoes.com.br`

---

## O que a API entrega

- **Questões classificadas** por matéria, assunto (tópicos em árvore), banca, instituição, cargo, área, âmbito, ano, nível de escolaridade e dificuldade.
- **Filtros combináveis**: todos opcionais, e os plurais aceitam vários valores (`examiningBoardIds=1,2` traz questões das duas bancas).
- **Busca textual** no enunciado e por trecho do nome do cargo.
- **Provas completas**: liste as provas e puxe todas as questões de uma delas, para montar simulados fiéis ao concurso.
- **Questões anuladas e desatualizadas sinalizadas**: dá para excluir da lista as anuladas pela banca e as que cobram uma norma que mudou depois da prova.
- **Gabarito sob demanda**: a listagem traz enunciado e alternativas, e o gabarito vem num endpoint próprio, por questão, na hora em que o aluno responde.
- **Fórmulas e imagens**: enunciados com LaTeX (renderizável com KaTeX) e imagens hospedadas em CDN.
- **Questões próprias** (planos Pro e Business): cadastre questões suas no painel e consuma pela mesma API, misturadas com as do acervo.
- **Reporte de erro**: seus alunos podem apontar uma questão com problema direto pela API.
- **Dois contratos**: `v1` com campos em português e `v2` com campos em inglês. Mesmo acervo, mesma consulta, você escolhe o idioma.

---

## Como usar

### 1. Crie uma conta e pegue sua chave

Cadastre-se pelo site (e-mail ou Google). O plano **Sandbox** é gratuito e serve um conjunto fixo de questões de demonstração, o bastante para construir a integração inteira contra a API real. Quando quiser o acervo completo, há **teste grátis de 7 dias sem cartão** e depois os planos pagos.

A chave de API aparece no painel do usuário.

### 2. Faça a primeira chamada

Toda requisição leva a chave no header `x-api-key`:

```bash
curl "https://api.apidasquestoes.com.br/api/v2/questions?size=5&years=2024&nullified=false" \
  -H "x-api-key: SUA_CHAVE"
```

Buscar o gabarito de uma questão:

```bash
curl "https://api.apidasquestoes.com.br/api/v2/questions/42/answer" \
  -H "x-api-key: SUA_CHAVE"
```

### 3. Principais endpoints

| Método | Endpoint (v2) | O que faz |
|---|---|---|
| `GET` | `/api/v2/questions` | Lista questões com filtros e paginação |
| `GET` | `/api/v2/questions/{id}` | Uma questão |
| `GET` | `/api/v2/questions/{id}/answer` | Gabarito da questão |
| `GET` | `/api/v2/questions/jobs` | Cargos disponíveis |
| `GET` | `/api/v2/exams` | Provas disponíveis |
| `GET` | `/api/v2/exams/{id}/questions` | Questões de uma prova |

O equivalente em português fica em `/api/v1` (`/questoes`, `/questoes/{id}/gabarito`, `/materias`, `/bancas`, `/instituicoes`, `/areas`, `/ambitos`, `/topicos`). A lista completa de parâmetros está na [documentação](https://www.apidasquestoes.com.br/documentacao).

### 4. Exemplos de uso

- **Resolução de questões**: liste por matéria e banca, mostre enunciado e alternativas, e consulte o gabarito quando o aluno marcar a resposta.
- **Simulado de uma prova real**: `GET /api/v2/exams` para achar a prova, `GET /api/v2/exams/{id}/questions` para montar o caderno.
- **Simulado personalizado**: combine `subjectIds`, `examiningBoardIds`, `levels` e `difficulties` para montar listas sob medida para cada aluno.
- **Questão do dia**: um fluxo no n8n que puxa uma questão por dia e envia por e-mail, WhatsApp ou Telegram.
- **Front-end direto no navegador**: cadastre o domínio do seu site no painel e chame a API pelo JavaScript do cliente; requisições de outros domínios com a sua chave são recusadas.
- **Back-end próprio**: chame do seu servidor, guarde o que fizer sentido em cache e entregue para o seu app.

### Limites

- Até 25 questões por página (o valor efetivo volta na resposta).
- Limite de requisições por minuto por cliente.
- Cota mensal de questões conforme o plano. Os valores atualizados estão em [Planos](https://www.apidasquestoes.com.br/planos).

---

## Stack

### Back-end (este repositório)

| Tecnologia | Uso no projeto |
|---|---|
| **Java 17** + **Spring Boot 3.5** | Base da API REST (Web, Validation, Data JPA, Security, Mail, Actuator) |
| **PostgreSQL** | Banco principal do acervo, usuários e assinaturas |
| **Redis** | Cache das consultas, rate limit por minuto e contadores de cota |
| **RabbitMQ** | Registro assíncrono das requisições, sem atrasar a resposta |
| **Spring Security** + **JWT (jjwt)** | Autenticação do painel; chave de API (`x-api-key`) para clientes |
| **Login com Google** (OAuth2 / ID token) | Cadastro e login sem senha |
| **Mercado Pago SDK** | Assinaturas recorrentes e webhooks de pagamento |
| **Resend** (SMTP + webhooks) | E-mails transacionais e acompanhamento de entrega |
| **Cloudflare R2** (AWS SDK S3) | Armazenamento das imagens das questões |
| **Anthropic Claude** (SDK Java) | Apoio à importação e classificação de questões no painel admin |
| **jsoup** | Saneamento do HTML dos enunciados |
| **MapStruct** + **Lombok** | Mapeamento entre entidades e DTOs, menos boilerplate |
| **Micrometer** + **Prometheus** | Métricas de latência e uso |
| **Thymeleaf** | Templates dos e-mails |
| **JUnit** + **H2** | Testes automatizados |
| **Docker** | Build em duas etapas e imagem de runtime enxuta (JRE 17 Alpine) |

### Front-end ([repositório](https://github.com/danielsfilho/api-de-questoes-front-end))

| Tecnologia | Uso no projeto |
|---|---|
| **React 19** + **Vite** | Site, painel do cliente e painel admin |
| **React Router 7** | Rotas públicas e áreas autenticadas |
| **Tailwind CSS 4** | Estilização |
| **Axios** | Chamadas à API |
| **KaTeX** | Renderização de fórmulas nos enunciados |
| **Tiptap** | Editor rico para cadastrar e revisar questões |
| **pdf.js** | Leitura de provas em PDF na importação |
| **lucide-react** | Ícones |
| **Nginx** + **Docker** | Servir o build em produção |

---

## Contato

Dúvidas, parcerias ou integrações sob medida: pelo [site](https://www.apidasquestoes.com.br) ou abrindo uma issue neste repositório.

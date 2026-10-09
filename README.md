# MoneyFlow - Gestão Financeira e Investimentos

[![Status](https://img.shields.io/badge/status-em_desenvolvimento-yellow)]()
[![Versão](https://img.shields.io/badge/versão-0.1.0-blue)]()
[![Licença](https://img.shields.io/badge/licença-acadêmica-lightgrey)]()

**Instituição:** Centro Universitário de Brasília (UniCEUB)
**Curso:** Ciências da computação
**Disciplina:** Desenvolvimento Web  
**Turma / Semestre:** 2026.4  
**Professor(a):** Felippe Pires Ferreira  
**Status do projeto:** Protótipo

---

## Sumário

- [1. Descrição do projeto](#1-descrição-do-projeto)
- [2. Funcionalidades](#2-funcionalidades)
- [3. Demonstração](#3-demonstração)
- [4. Tecnologias utilizadas](#4-tecnologias-utilizadas)
- [5. Arquitetura](#5-arquitetura)
- [6. Organização dos diretórios](#6-organização-dos-diretórios)
- [7. Participantes](#7-participantes)
- [8. Como executar](#8-como-executar)
- [9. Configuração](#9-configuração)
- [10. Testes](#10-testes)
- [11. Uso de inteligência artificial](#11-uso-de-inteligência-artificial)
- [12. Contribuição e fluxo de trabalho](#12-contribuição-e-fluxo-de-trabalho)
- [13. Histórico de versões](#13-histórico-de-versões)
- [14. Limitações e próximos passos](#14-limitações-e-próximos-passos)
- [15. Licença, referências e contato](#15-licença-referências-e-contato)

---

## 1. Descrição do projeto

O MoneyFlow é uma aplicação web de gestão financeira pessoal e acompanhamento de carteira de investimentos. O problema central que o sistema resolve é a fragmentação de dados: pequenos investidores frequentemente precisam utilizar planilhas complexas ou múltiplos aplicativos distintos para controlar o seu fluxo de caixa diário e, simultaneamente, acompanhar a rentabilidade da sua carteira de ativos (renda fixa, ações e criptomoedas). A solução proposta centraliza essas vertentes em um único dashboard intuitivo.

### Objetivos

**Objetivo geral:** Desenvolver uma aplicação web com backend em Python e Django para centralizar a gestão de finanças pessoais e investimentos, com atualização de dados de mercado via consumo de API REST externa.

**Objetivos específicos:**
* Permitir o cadastro seguro, autenticação e controle de acesso de múltiplos usuários.
* Gerenciar o CRUD completo de Contas Bancárias, Categorias, Transações e Ordens de Investimentos.
* Integrar o sistema a uma API financeira pública para atualizar cotações.
* Fornecer uma API REST própria para consulta de dados, preparada para integrações futuras (como um bot do Telegram, fora do escopo desta versão).

### Público-alvo
* Jovens investidores buscando consolidar fluxo de caixa e carteira de ativos.
* Pessoas físicas focadas em organização financeira pessoal a longo prazo.

---

## 2. Funcionalidades

| Funcionalidade | Descrição | Status |
| :--- | :--- | :--- |
| **Autenticação** | Login, logout e isolamento de dados por usuário | Planejada |
| **Cadastros Base** | Gestão de Contas Bancárias e Categorias personalizadas | Planejada |
| **Transações** | Lançamento de receitas e despesas no fluxo de caixa | Planejada |
| **Investimentos** | Lançamento de ordens de compra e venda de ativos | Planejada |
| **Relatórios** | Consolidação do histórico patrimonial (caixa + investimentos) | Planejada |
| **Integração Externa** | Consumo de API para cotações de ativos em tempo real | Planejada |
| **API REST Própria** | Endpoints em JSON para consulta de dados financeiros | Planejada |

### Requisitos não funcionais

* **Desempenho:** Respostas da API em menos de 2 segundos.
* **Segurança:** Senhas armazenadas com hash.
* **Usabilidade:** Interface responsiva construída com react.
* **Disponibilidade:** Aplicação hospedada e acessível via URL pública com HTTPS (Fase 2).

---

## 3. Demonstração

*Inclua capturas de tela, GIF ou link para vídeo. Coloque as imagens em `images/`.*

![Tela principal](images/[screenshot-principal].png)

| Tela | Descrição |
| --- | --- |
| [Login] | [Acesso ao sistema com e-mail e senha] |
| [Painel] | [Visão geral das reservas do dia] |

**Vídeo / protótipo:** [URL do YouTube, Loom ou Figma]

---

## 4. Tecnologias utilizadas

*Informe as tecnologias de fato usadas no projeto. Remova as linhas que não se aplicarem.*

| Camada | Tecnologia | Versão |
| --- | --- | --- |
| Linguagem | Python | a decidir |
| Frontend | HTML, CSS, React | a decidir |
| Backend | Django | a decidir |
| Banco de dados | PostgreSQL | a decidir |
| Testes | a decidir | a decidir |
| Infraestrutura | a decidir | a decidir |
| Outras ferramentas |Git, Figma, Postman, brModeloweb, Draw.io, Github | — |

---

## 5. Arquitetura

A solução segue uma arquitetura cliente-servidor em camadas: uma SPA em React (frontend) consome uma API REST própria, desenvolvida com Django REST Framework (backend), que persiste os dados em PostgreSQL e consome uma API externa de cotações. O backend é uma aplicação única (monolito); a separação entre frontend e backend permite evoluir cada lado de forma independente.

![Diagrama de componentes do MoneyFlow](docs/arquitetura/Diagrama%20-%20arquitetura.png)

**Fluxo de Dados:**
`[Navegador / React] ⇄ [API REST / Django] ⇄ [Banco de Dados / PostgreSQL]`

Em paralelo para as cotações:
`[API REST / Django] → Request HTTP → [API Externa]`

**Decisões relevantes:**
* **Separação frontend/backend (React + Django REST):** a API REST separa cliente e servidor, permitindo trocar ou evoluir o frontend sem alterar as regras de negócio.
* **Cotações sob demanda:** o cálculo do patrimônio (histórico) busca os preços atualizados em uma API externa, então o sistema não precisa armazenar cotações diárias.
* **Tolerância a falhas da API externa:** as chamadas usam timeout e tratamento de erro, para que a indisponibilidade do provedor não derrube o restante da aplicação.

### Endpoints principais (quando houver API)

| Método | Rota | Descrição |
| --- | --- | --- |
| `POST` | `/api/[recurso]` | [Ex.: criar um registro] |
| `GET` | `/api/[recurso]` | [Ex.: listar registros] |
| `GET` | `/api/[recurso]/{id}` | [Ex.: obter um registro] |
| `PUT` | `/api/[recurso]/{id}` | [Ex.: atualizar um registro] |
| `DELETE` | `/api/[recurso]/{id}` | [Ex.: remover um registro] |

Documentação completa da API: [link para Swagger, Postman ou `docs/api.md`]

---

## 6. Organização dos diretórios

```text
.
├── README.md                 # Documentação principal do projeto
├── .env.example              # Modelo de variáveis de ambiente (sem segredos)
├── docs/                     # Modelagem do projeto e artefatos técnicos
│   ├── README.pdf            # Índice da pasta docs/
|   ├── api/                  # Documentação da API
|   |   ├── README.md
|   ├── arquitetura/          # Diagramas de arquitetura do sistema
|   |   ├── Diagrama - arquitetura.drawio
|   |   ├── Diagrama - arquitetura.png
│   ├── modelagem/ 
│   |   ├── banco-de-dados/   # Artefatos do banco de dados
|   |   |   ├── README.md
│   |   |   ├── diagrama-er.pdf
|   |   |   ├── dicionario_de_dados.pdf
|   |   |   ├── dicionario_de_dados.xlsx
|   |   |   ├── modelagem_conceitual.pdf
|   |   |   ├── modelagem_logica.pdf
│   |   |   └── modelo-logico.pdf
│   |   ├── casos-de-uso/      # Especificações e diagramas de casos de uso
│   |   │   └── especificacoes-casos-de-uso.pdf
│   |   └── classes/           # Diagramas de classe
|   |       ├── Diagrama_classe.drawio
|   |       ├── Diagrama_classe.png
|   |       └── Diagrama-de-classes.pdf
|   |
|   |
|   ├── prototipo/              # Link ao protótipo de interface
|   |   └── README.md
|   └── visao/                  # Documento de visão do projeto
|       └──documento-de-visão
|      
├── images/                   # Figuras da documentação geral
├── src/                      # Código-fonte da aplicação
│   ├── frontend/             # Interface com o usuário (quando houver)
│   └── backend/              # Regras de negócio, API e acesso a dados (quando houver)
├── tests/                    # Testes automatizados
└── scripts/                  # Scripts auxiliares de setup, build ou deploy
```

| Diretório / arquivo | Função |
| --- | --- |
| `README.md` | Apresentação do projeto, objetivos, tecnologias e instruções de uso |
| `.env.example` | Lista das variáveis necessárias, sem credenciais reais |
| `docs/` | Artefatos de análise, visão do produto e documentação da API |
| `docs/arquitetura/` Diagramas arquiteturais da aplicação|
| `docs/modelagem/` | Casos de uso, classes e modelo de dados (ER e lógico) |
| `images/` | Figuras e ilustrações para a documentação geral do repositório |
| `src/` | Código-fonte organizado por camada (frontend e backend) e módulo. |
| `tests/` | Casos de teste e evidências de verificação |
| `scripts/` | Automação de ambiente e execução |

---

## 7. Participantes

| Nome | Matrícula | Função no projeto |
| --- | --- | --- |
| Nicolas Klaczko Hogan | 22506264 | coordenação / full-stack / testes / documentação |
| Gabriel Soares Segatto | 22502904 | coordenação / full-stack / testes / documentação |
| André Yuri Alves Silva  | 22509843 | coordenação / full-stack / testes / documentação |


**Professor(a) responsável:** Felippe Pires Ferreira
---

## 8. Como executar

*Preencha com os comandos reais do projeto para que outra pessoa consiga reproduzir o ambiente.*

### Pré-requisitos

- [Ex.: Git]
- [Ex.: Python 3.12+]
- [Ex.: Node.js 20+]
- [Ex.: Docker]

### Instalação e execução

```bash
# 1. Clonar o repositório
git clone [https://github.com/GabrielSSegatto/Gerenciador_financas.git]
cd [NOME_DA_PASTA]

# 2. Instalar dependências
[comando de instalação]

# 3. Configurar variáveis de ambiente
cp .env.example .env
# edite o arquivo .env com as credenciais locais

# 4. Executar a aplicação
[comando de execução]
```

**Acesso local:** [Ex.: http://localhost:3000]

### Implantação (quando houver)

- **Ambiente:** [Ex.: Render, Railway, Vercel, servidor da instituição]
- **URL de produção:** [https://...] [https://github.com/GabrielSSegatto/Gerenciador_financas.git]
- **Observações:** [Ex.: é necessário configurar as variáveis de ambiente no painel do provedor]

---

## 9. Configuração

*Liste as variáveis de ambiente usadas pelo sistema. Nunca publique senhas, tokens ou chaves neste arquivo.*

| Variável | Obrigatória | Descrição | Exemplo |
| --- | --- | --- | --- |
| `PORT` | Sim | Porta da aplicação | `3000` |
| `DATABASE_URL` | Sim | Conexão com o banco | `postgresql://user:senha@localhost:5432/app` |
| `SECRET_KEY` | Sim | Chave de sessão / JWT | `[gerar localmente]` |

Credenciais reais devem ficar apenas no arquivo `.env` (não versionado).

---

## 10. Testes

*Descreva como executar os testes e o que eles cobrem.*

```bash
[comando para executar os testes]
```

| Tipo | Ferramenta | O que verifica |
| --- | --- | --- |
| Unitários | [Ex.: pytest / JUnit / Jest] | [Ex.: regras de negócio isoladas] |
| Integração | [Ex.: ...] | [Ex.: API e banco de dados] |
| Manuais | [Ex.: checklist em `docs/`] | [Ex.: fluxos principais da interface] |

**Cobertura atual:** [Ex.: 70% / não medida]

---

## 11. Uso de inteligência artificial

Este repositório segue a política de uso de IA da disciplina (semáforo pedagógico):

![Política de uso de IA — semiformal](images/semaforo.png)

| Situação | Significado |
| --- | --- |
| **Vermelho — uso proibido** | Atividades de autonomia intelectual (ex.: provas presenciais sem consulta). |
| **Amarelo — uso limitado** | IA pode ser ferramenta auxiliar, desde que haja declaração de uso. |
| **Verde — uso permitido** | Uso livre ao longo da atividade acadêmica. |

### Declaração de uso

*Preencha de forma honesta. Se não houve uso de IA, declare explicitamente.*

- **Houve uso de IA neste projeto?** Sim
- **Ferramentas utilizadas:** ChatGPT, Gemini
- **Finalidade:** Finalidade: Auxílio na formatação de textos (Markdown), validação de diagramas, esclareciemento de duvidas de sintaxe, auxilio na prototipação do front end com a IA do Figma
- **O que NÃO foi delegado à IA:** decisões arquiteturais, definição do escopo e regras do negócio
---

## 12. Contribuição e fluxo de trabalho

*Padronize o trabalho em equipe. Ajuste as regras ao combinado da disciplina.*

### Branches

- `main` — versão estável para avaliação
- `feat/[nome]` — nova funcionalidade
- `fix/[nome]` — correção de defeito
- `docs/[nome]` — alterações só de documentação

### Commits

Use mensagens curtas e no imperativo, por exemplo:

- `feat: adiciona cadastro de reservas`
- `fix: corrige validação de data`
- `docs: atualiza instruções de execução`

### Passos sugeridos

1. Criar uma branch a partir de `main`.
2. Implementar e testar localmente.
3. Abrir um *pull request* / *merge request* para revisão do grupo.
4. Só então integrar à branch principal.

**Issues e quadro de tarefas:** [link do GitHub Projects, Trello ou similar]

---

## 13. Histórico de versões

*Registre entregas relevantes (sprints, checkpoints ou versões avaliadas).*

| Versão | Data | Descrição |
| --- | --- | --- |
| `x.x.x` | [AAAA-MM-DD] | [Ex.:...  |
| `0.0.1` | 2026-10-08 | estrutura inicial do repositório |

---

## 14. Limitações e próximos passos

### Problemas conhecidos

- [Ex.: a recuperação de senha ainda não envia e-mail]
- [Ex.: o layout quebra em telas menores que 360 px]

### Roadmap

- [ ] [Ex.: autenticação com dois fatores]
- [ ] [Ex.: exportação de relatórios em CSV]
- [ ] [Ex.: implantação em ambiente de homologação]

---

## 15. Licença, referências e contato

**Licença:** [Ex.: uso exclusivamente acadêmico / MIT / outro]

Este material destina-se a fins educacionais. Verifique com a disciplina se o código pode ser reutilizado fora do curso.

### Documentação complementar

- Índice da pasta `docs/`: [`docs/README.pdf`](docs/README.pdf)
- Casos de uso (diagrama + especificações): [`docs/modelagem/casos-de-uso/especificacoes-casos-de-uso.pdf`](docs/modelagem/casos-de-uso/especificacoes-casos-de-uso.pdf)
- Diagrama de classes: [`docs/modelagem/classes/diagrama-de-classes.pdf`](docs/modelagem/classes/diagrama-de-classes.pdf)
- Modelo conceitual (ER): [`docs/modelagem/banco-de-dados/diagrama-er.pdf`](docs/modelagem/banco-de-dados/diagrama-er.pdf)
- Modelo lógico: [`docs/modelagem/banco-de-dados/modelo-logico.pdf`](docs/modelagem/banco-de-dados/modelo-logico.pdf)
- Apresentação: [`docs/apresentacao.pdf`](docs/)

### Referências

- [Autor. Título. Ano. URL ou dados bibliográficos.]
- [Documentação oficial da tecnologia X.]

### Contato

Dúvidas sobre o projeto: [e-mail institucional do grupo ou issue no repositório]

**Agradecimentos:** Felippe Pires Ferreira

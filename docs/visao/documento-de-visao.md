# Documento de Visão — MoneyFlow

| | |
| --- | --- |
| **Projeto** | MoneyFlow — Gestão Financeira e Investimentos |
| **Status** | Em desenvolvimento |

---

## 1. Propósito do documento

Este documento define a visão de alto nível do sistema MoneyFlow: o problema que ele se propõe a resolver, o perfil dos usuários, as principais funcionalidades, as restrições e os riscos iniciais. O objetivo é manter o alinhamento entre as decisões de projeto e os requisitos do sistema ao longo do desenvolvimento.

## 2. Contexto e descrição do problema

Atualmente, investidores de varejo e pessoas físicas utilizam plataformas separadas para gerenciar o próprio dinheiro: aplicativos bancários e planilhas para o fluxo de caixa do dia a dia (receitas e despesas) e home brokers ou corretoras para acompanhar investimentos em bolsa (B3), criptomoedas e renda fixa.

Essa fragmentação dificulta enxergar o patrimônio total e a relação entre o que se gasta e o que se investe. O MoneyFlow resolve esse problema ao unificar a gestão financeira tradicional e o acompanhamento de ativos em um único painel consolidado.

## 3. Justificativa

Quem investe em pequena escala costuma dividir a vida financeira entre planilhas, aplicativos bancários e corretoras, o que torna difícil acompanhar a evolução do patrimônio. Unificar fluxo de caixa e carteira em um só lugar reduz esse esforço e oferece uma visão consolidada, atualizada com cotações de mercado e com histórico diário da evolução patrimonial.

## 4. Objetivos

**Objetivo geral:** desenvolver uma aplicação web que centralize a gestão de finanças pessoais e o acompanhamento de investimentos, com cotações atualizadas por APIs externas.

**Objetivos específicos:**

- Permitir cadastro, autenticação e isolamento dos dados de cada usuário.
- Gerenciar contas bancárias, categorias, transações e ordens de investimento.
- Calcular o patrimônio consolidado (caixa + carteira de investimentos) com cotações atualizadas.
- Registrar automaticamente um histórico diário da evolução patrimonial.
- Disponibilizar uma API REST própria para consulta dos dados, preparada para integrações futuras.

## 5. Público-alvo

- Jovens investidores que querem consolidar o fluxo de caixa e a carteira de ativos.
- Pessoas físicas focadas em organização financeira pessoal de longo prazo.

## 6. Perfil dos envolvidos (stakeholders)

| Perfil | Descrição e responsabilidades |
| --- | --- |
| Usuário final | Pessoa física que deseja organizar as finanças pessoais, acompanhar os gastos por categoria e gerenciar a carteira de investimentos em um só lugar. |
| Desenvolvedor | Responsável pela modelagem, implementação (React e Django), testes, documentação e publicação da aplicação. |
| Provedores de dados (APIs) | Plataformas externas (Brapi e CoinGecko) que fornecem as cotações dos ativos de mercado, essenciais para o cálculo do patrimônio. |
| Consumidores futuros da API | Integrações que poderão consumir a API REST própria, como um bot do Telegram (fora do escopo desta versão). |

## 7. Escopo do produto (o que o sistema faz)

O MoneyFlow será uma Single Page Application (SPA) responsiva, com as seguintes funcionalidades principais:

- **Autenticação e segurança:** cadastro de usuários e login seguro, com senhas armazenadas em formato criptografado (hash) e isolamento dos dados por usuário.
- **Gestão de fluxo de caixa:** cadastro de contas bancárias, criação de categorias personalizadas e lançamento de receitas e despesas.
- **Gestão de carteira:** registro de ordens de compra e venda de ativos (Ações, FIIs, Criptomoedas e Renda Fixa). Os ativos ficam em um catálogo global; a posição de cada usuário é obtida a partir de suas ordens.
- **Sincronização de mercado:** consumo de APIs externas (Brapi e CoinGecko) para atualizar o preço dos ativos de renda variável e criptomoedas no momento da consulta do usuário. O valor da Renda Fixa é estimado pelo sistema a partir do indexador e da taxa de rendimento informados no cadastro do ativo.
- **Consolidação patrimonial:** cálculo do valor total em caixa somado ao valor atualizado da carteira de investimentos.
- **Histórico patrimonial diário:** o sistema registra, uma vez por dia, um *snapshot* do patrimônio de cada usuário (total em caixa e total investido), permitindo acompanhar a evolução ao longo do tempo.
- **API REST própria:** exposição dos dados financeiros em JSON para consulta, projetada para ser consumida por integrações futuras (como um bot do Telegram).

### Requisitos não funcionais

- **Desempenho:** respostas da API em menos de 2 segundos nas consultas principais.
- **Segurança:** senhas armazenadas com hash; acesso restrito aos dados do próprio usuário; conexão segura (HTTPS) em produção.
- **Usabilidade:** interface responsiva, construída com React.
- **Resiliência:** falhas nas APIs externas não podem impedir o uso das demais funcionalidades.

## 8. Fora do escopo (o que o sistema não faz)

Para delimitar o tamanho do projeto, as seguintes funcionalidades não serão implementadas nesta versão:

- **Transações financeiras reais:** o sistema não atua como instituição de pagamento (não faz PIX, TED nem pagamento de boletos).
- **Execução de ordens no mercado:** o sistema não é uma corretora; não efetua compras reais de ações na B3 nem de criptomoedas em exchanges. Os registros são apenas espelhos virtuais para controle pessoal.
- **Integração com Open Finance:** não haverá leitura automática de extratos de outras instituições; todos os lançamentos são manuais.
- **Consultoria financeira:** o sistema não fornecerá recomendações automatizadas de compra ou venda com base no perfil do investidor.
- **Bot do Telegram:** a API REST própria foi pensada para essa integração futura, mas o bot não será desenvolvido neste projeto.
- **Aplicativos nativos:** não haverá aplicativos para Android ou iOS; o acesso é feito pelo navegador.

## 9. Restrições

- **Arquiteturais:** o frontend é desenvolvido em React e o backend em Django REST Framework, comunicando-se exclusivamente via JSON. O banco de dados relacional é o PostgreSQL.
- **Dependência externa:** o cálculo do patrimônio investido em renda variável e criptomoedas depende da disponibilidade das APIs da Brapi e do CoinGecko. O backend deve implementar timeouts e tratamento de erros para que a indisponibilidade desses provedores não derrube o sistema, permitindo ao usuário acessar ao menos o fluxo de caixa em caso de falha.
- **Ambiente de uso:** o sistema é web (acessível pelo navegador), sem aplicações nativas nesta fase inicial.

## 10. Premissas

- As APIs externas (Brapi e CoinGecko) continuarão disponíveis, com plano gratuito suficiente para o uso previsto.
- Os lançamentos de receitas, despesas e ordens são feitos manualmente pelo usuário.
- O usuário acessa o sistema por um navegador, com conexão à internet.
- Existe um mecanismo de agendamento capaz de executar o snapshot patrimonial diário.

## 11. Riscos iniciais

| Risco | Impacto | Mitigação |
| --- | --- | --- |
| Indisponibilidade ou limite de requisições das APIs externas | Patrimônio investido sem cotação atualizada | Timeout, tratamento de erros e uso do fluxo de caixa mesmo com a falha |
| Falha na rotina diária do snapshot (por exemplo, API fora do ar no horário) | Dia sem registro no histórico ou valores incorretos | Nova tentativa, registro da falha e não gravar valores incompletos |
| Exposição de dados financeiros | Perda de confiança e vazamento de dados | Senhas com hash, isolamento por usuário, HTTPS e segredos fora do repositório |
| Inconsistência entre saldo e transações | Valores incorretos exibidos ao usuário | Atualizar o saldo e registrar a transação na mesma operação (transação atômica) |
| Aumento do escopo | Projeto não concluído | Manter a lista de "Fora do escopo" e priorizar o essencial |

## 12. Critérios de sucesso

- O usuário consegue cadastrar contas, categorias e transações e ver o saldo correto.
- O usuário consegue registrar ordens de compra e venda de ações, FIIs, criptomoedas e renda fixa.
- O patrimônio consolidado (caixa + investimentos) é calculado com cotações atualizadas.
- O sistema registra um snapshot patrimonial por dia, sem duplicidade para o mesmo usuário e a mesma data.
- Uma falha em uma API externa não impede o uso do fluxo de caixa.
- A API REST responde em menos de 2 segundos nas consultas principais.
- A documentação permite que outra pessoa execute o projeto localmente.

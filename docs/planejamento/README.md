# Planejamento e Execução — MoneyFlow

Este documento detalha a estratégia ágil de execução, os marcos de entrega, o backlog de tarefas e a gestão de riscos do projeto MoneyFlow. Para a Fase 2, adotamos Kanban, garantindo flexibilidade, autonomiano desenvolvimento.

## 1. Estratégia de Execução (Kanban)
* **Sistema *Pull* (Sob Demanda):** As tarefas não possuem um "dono" pré-definido. Como os três membros atuam como Desenvolvedores Full-Stack, cada um escolhe e puxa as tarefas do topo do backlog conforme a sua disponibilidade e afinidade técnica no momento.
* **Fatias Verticais:** Cada tarefa deve entregar uma funcionalidade completa (Backend + Frontend + Testes), evitando criar "gargalos" entre quem faz a API e quem faz a tela.
* **Integração Contínua (CI):** A qualidade é garantida via GitHub Actions, executando os testes automatizados a cada *Pull Request*.

## 2. Fluxo de Trabalho e Quadro Kanban
O gerenciamento visual das tarefas será feito no GitHub Projects (ou Trello), com as seguintes colunas e regras de **WIP (Work In Progress)**:

1. **Backlog (To Do):** Lista de todas as tarefas mapeadas e priorizadas.
2. **Selecionado (Next):** Tarefas priorizadas para a semana atual.
3. **Em Andamento (WIP):** O que está sendo codificado agora. *Regra de Limite de WIP:* Máximo de 1 a 2 tarefas por membro simultaneamente para garantir que as coisas sejam terminadas antes de novas serem iniciadas.
4. **Em Revisão (Review/PR):** Código finalizado aguardando aprovação (Pull Request) de outro membro do grupo.
5. **Feito (Done):** Código mergeado na branch `main`.

## 3. Marcos 
Embora a puxada de tarefas seja livre, o grupo focará em esgotar os itens de um Marco antes de puxar itens do próximo, garantindo a entrega do MVP:

| Marco | Entregas Foco | Status |
| :--- | :--- | :--- |
| **M0 — Fundação** | Organização do Repositório, Documentos, Modelagem de BD e Contratos de API. | **Concluído** ✅ |
| **M1 — Autenticação e Base** | Configuração do JWT, Login/Registro, Contas Bancárias e Categorias. | A Iniciar |
| **M2 — Transações (MVP)**| Lançamentos financeiros, cálculo dinâmico de saldo e filtros. | A Iniciar |
| **M3 — Investimentos** | Consumo Brapi/CoinGecko, Ordens de compra/venda e posições. | A Iniciar |
| **M4 — Patrimônio** | Dashboard consolidado e rotina de snapshot patrimonial. | A Iniciar |

## 4. Backlog de Tarefas 
*(Prioridade: A = Alta/Bloqueante; M = Média; B = Baixa)*

**Autenticação e Base**
* [A] Criar modelo de Utilizador customizado no Django e configurar JWT
* [A] Desenvolver telas de Login e Registro no React
* [A] Implementar CRUD de Contas Bancárias (API + Frontend)
* [A] Implementar CRUD de Categorias (API + Frontend)

**Transações (Fluxo de Caixa)**
* [A] Criar modelo e endpoints de Transações
* [A] Desenvolver tela de Transações com modal de novo registro (React)
* [A] Criar *trigger*/lógica atômica para atualizar saldo da Conta ao alterar transação
* [M] Criar filtros de busca (por data, categoria, conta)

**Investimentos e Cotações**
* [A] Implementar cliente HTTP para a Brapi e CoinGecko com *timeout* e *fallback*
* [A] CRUD de Ordens de Investimento e validação de quantidade
* [A] Desenvolver tela da Carteira de Investimentos e modal de Ordem
* [M] Implementar cache curto no backend para cotações (evitar *rate limit*)

**Patrimônio e Dashboard**
* [A] Endpoint de consolidação matemática (Caixa + Investimentos atualizados)
* [A] Desenvolver o Dashboard principal no React (Gráficos e Indicadores)
* [A] Deploy final (Backend, Frontend e Banco de Dados)
* [M] Rotina de snapshot diário do patrimônio (Cron job)

## 5. Riscos do Projeto e Plano de Mitigação

| Risco | Plano de Mitigação |
| :--- | :--- |
| **Instabilidade / Limites das APIs Gratuitas** | Uso de *cache* no banco de dados e *timeouts*. Se a API falhar, o sistema exibirá o último preço conhecido para não travar a aplicação. |
| **Conflitos de Código (Merge Conflicts)** | Comunicação constante no grupo. O uso de Fatias Verticais isola o trabalho de cada membro em módulos diferentes. |
| **Tempo / Atraso na Entrega** | Caso o prazo aperte, tarefas de prioridade [M] (como filtros avançados e snapshot diário automático) serão cortadas para garantir a entrega do MVP. |

## 6. Decisões em Aberto
* **Hospedagem (Deploy):** Avaliar infraestruturas em nuvem com melhor custo-benefício estudantil (ex: Vercel para React; Render ou Railway para o Django/PostgreSQL).
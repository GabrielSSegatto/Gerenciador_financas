# Documentação de APIs e Integrações

## 1. Contrato Inicial da API (Interna - MoneyFlow)
A API RESTful do MoneyFlow permite a comunicação entre o frontend (React) e o backend (Django REST). A autenticação é efetuada via token JWT, transmitido no cabeçalho: `Authorization: Bearer <token>`.

### 1.1. Registar Transação (Receita/Despesa)
*   **Método e Rota:** `POST /api/transacoes/`
*   **Descrição:** Regista uma nova movimentação financeira no fluxo de caixa.
*   **Parâmetros (Corpo da Requisição JSON):**
    ```json
    {
      "id_conta_bancaria": 1,
      "id_categoria": 3,
      "valor": 150.50,
      "descricao": "Compra de material",
      "data": "2026-10-08T20:00:00Z"
    }
    ```
*   **Respostas Esperadas e Códigos de Status:**
    *   `201 Created`: Transação registada com sucesso.
        ```json
        { "id_transacao": 42, "status": "sucesso" }
        ```
    *   `400 Bad Request`: Erro de validação (ex: campos obrigatórios ausentes).
    *   `401 Unauthorized`: Token JWT inválido ou ausente.

### 1.2. Consultar Histórico Patrimonial
*   **Método e Rota:** `GET /api/patrimonio/`
*   **Descrição:** Retorna o valor consolidado (caixa + carteira de investimentos atualizada).
*   **Parâmetros (Query):** Nenhum.
*   **Respostas Esperadas e Códigos de Status:**
    *   `200 OK`:
        ```json
        {
          "data_consulta": "2026-10-08",
          "total_caixa": 5400.00,
          "total_investido": 12350.75,
          "patrimonio_liquido": 17750.75
        }
        ```

---

## 2. Plano de Integração Externa
O backend do MoneyFlow consome dados de mercado em tempo real para calcular a valorização da carteira do utilizador.

### 2.1. Brapi (Ações e FIIs da B3)
*   **Finalidade:** Obter as cotações atualizadas de ativos negociados na bolsa brasileira.
*   **Endpoint Consumido:** `GET https://brapi.dev/api/quote/{ticker}`
*   **Dados Utilizados:** O campo `regularMarketPrice` presente no retorno JSON.
*   **Autenticação e Limites:** Requer token passado na query string (`?token=SEU_TOKEN`). O plano gratuito suporta o volume necessário para a Fase 1 e testes.
*   **Documentação Oficial:** [https://brapi.dev/docs](https://brapi.dev/docs)

### 2.2. CoinGecko (Criptomoedas)
*   **Finalidade:** Obter as cotações em tempo real de ativos digitais (ex: Bitcoin).
*   **Endpoint Consumido:** `GET https://api.coingecko.com/api/v3/simple/price?ids={moeda}&vs_currencies=brl`
*   **Dados Utilizados:** O valor da moeda convertido para BRL (`brl`).
*   **Autenticação e Limites:** API pública. Limite variável entre 10 a 50 requisições por minuto no plano base.
*   **Documentação Oficial:** [https://docs.coingecko.com/](https://docs.coingecko.com/)

### 2.3. Tratamento de Indisponibilidade (Fallback)
Caso alguma das APIs externas retorne erros (ex: `503 Service Unavailable`, `429 Too Many Requests`) ou exceda o tempo limite de resposta (*timeout* configurado para 5 segundos):
1. O sistema não bloqueará o acesso do utilizador e as funcionalidades de fluxo de caixa continuarão a operar normalmente.
2. O cálculo de investimentos utilizará o **último preço válido** guardado em cache no banco de dados.
3. Será emitido um aviso no frontend: *"Aviso: Cotações de mercado desatualizadas por instabilidade no provedor de dados."*
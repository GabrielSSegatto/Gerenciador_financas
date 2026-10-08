# Modelagem do Banco de Dados 


* **`modelagem_conceitual.pdf`**: Modelo Conceitual
* **`modelagem_logica.pdf`**: Modelo Lógico
* **`dicionario-de-dados.xls`**: dicionário de dados em planilha
* **`dicionario-de-dados`**: dicionário de dados em PDF para consulta


---

## Decisão de Arquitetura: A redundância na tabela de Transações

A existência de usuario_id em transação é uma redundância e fere a 3ª Forma Normal. Porem, essa foi uma decisão consciente

Performance: para listar todas transações teria que passar pela tabela de contas, tornando a consulta mais pesada

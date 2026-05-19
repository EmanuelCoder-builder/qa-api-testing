# CASOS DE TESTE — API de Combustíveis V-Lab

**Versão:** 1.0  
**Data:** 2026-05-18  
**Autor:** Emanuel Henrique  

---

## Legenda

| Campo | Valores possíveis |
|-------|-------------------|
| **Prioridade** | Alta / Média / Baixa |
| **Tipo** | Funcional / Negócio / Segurança / Borda |
| **Status** | ✅ Passou / ❌ Falhou / ⏭️ Bloqueado / 🔲 Não executado |

---

## CT-001 — Smoke Test: API acessível

| Campo | Valor |
|-------|-------|
| **RN** | — |
| **Prioridade** | Alta |
| **Tipo** | Funcional |
| **Endpoint** | `GET /abastecimentos` |

**Pré-condição:** Nenhuma.  
**Passos:**
1. Enviar `GET https://v-lab-testes.onrender.com/abastecimentos`

**Resultado Esperado:** HTTP 200 com body contendo `data` (array) e `pagination`.  
**Status:** ✅

---

## CT-002 — CRUD: Criar abastecimento válido

| Campo | Valor |
|-------|-------|
| **RN** | — |
| **Prioridade** | Alta |
| **Tipo** | Funcional |
| **Endpoint** | `POST /abastecimentos` |

**Pré-condição:** `motorista_id` e `veiculo_id` válidos disponíveis.  
**Passos:**
1. Enviar POST com body completo e válido (todos os campos obrigatórios)

**Resultado Esperado:** HTTP 201 com objeto retornado contendo `id` gerado pelo backend.  
**Status:** ✅

---

## CT-003 — CRUD: Consultar abastecimento por ID

| Campo | Valor |
|-------|-------|
| **RN** | — |
| **Prioridade** | Alta |
| **Tipo** | Funcional |
| **Endpoint** | `GET /abastecimentos/:id` |

**Pré-condição:** CT-002 passou com sucesso.  
**Passos:**
1. Usar o `id` retornado no CT-002
2. Enviar `GET /abastecimentos/{id}`

**Resultado Esperado:** HTTP 200 com o objeto completo do abastecimento.  
**Status:** ✅

---

## CT-004 — CRUD: Atualizar abastecimento

| Campo | Valor |
|-------|-------|
| **RN** | — |
| **Prioridade** | Alta |
| **Tipo** | Funcional |
| **Endpoint** | `PUT /abastecimentos/:id` |

**Pré-condição:** CT-002 passou.  
**Passos:**
1. Enviar PUT com mesmo body do CT-002, mas com `litros: 55.0`

**Resultado Esperado:** HTTP 200 com objeto contendo `litros: 55.0`.  
**Status:** ❌

---

## CT-005 — CRUD: Deletar abastecimento

| Campo | Valor |
|-------|-------|
| **RN** | — |
| **Prioridade** | Alta |
| **Tipo** | Funcional |
| **Endpoint** | `DELETE /abastecimentos/:id` |

**Pré-condição:** CT-002 passou.  
**Passos:**
1. Enviar `DELETE /abastecimentos/{id}`
2. Em seguida, enviar `GET /abastecimentos/{id}`

**Resultado Esperado:** DELETE → HTTP 204 sem corpo; GET subsequente → HTTP 404.  
**Status:** ✅

---

## CT-006 — RN-01.1: `litros = 0` rejeitado

| Campo | Valor |
|-------|-------|
| **RN** | RN-01.1 |
| **Prioridade** | Alta |
| **Tipo** | Negócio |
| **Endpoint** | `POST /abastecimentos` |

**Passos:**
1. Enviar POST com `"litros": 0`

**Resultado Esperado:** HTTP 400 com mensagem de validação sobre `litros`.  
**Status:** ✅

---

## CT-007 — RN-01.1: `litros` negativo rejeitado

| Campo | Valor |
|-------|-------|
| **RN** | RN-01.1 |
| **Prioridade** | Alta |
| **Tipo** | Negócio |
| **Endpoint** | `POST /abastecimentos` |

**Passos:**
1. Enviar POST com `"litros": -10`

**Resultado Esperado:** HTTP 400 com mensagem de validação.  
**Status:** ✅

---

## CT-008 — RN-01.2: `valor_litro = 0` rejeitado

| Campo | Valor |
|-------|-------|
| **RN** | RN-01.2 |
| **Prioridade** | Alta |
| **Tipo** | Negócio |
| **Endpoint** | `POST /abastecimentos` |

**Passos:**
1. Enviar POST com `"valor_litro": 0`

**Resultado Esperado:** HTTP 400.  
**Status:** ❌

---

## CT-009 — RN-01.2: `valor_litro = 50.01` rejeitado (acima do limite)

| Campo | Valor |
|-------|-------|
| **RN** | RN-01.2 |
| **Prioridade** | Alta |
| **Tipo** | Borda |
| **Endpoint** | `POST /abastecimentos` |

**Passos:**
1. Enviar POST com `"valor_litro": 50.01`

**Resultado Esperado:** HTTP 400.  
**Status:** ❌

---

## CT-010 — RN-01.2: `valor_litro = 50.00` aceito (limite exato)

| Campo | Valor |
|-------|-------|
| **RN** | RN-01.2 |
| **Prioridade** | Alta |
| **Tipo** | Borda |
| **Endpoint** | `POST /abastecimentos` |

**Passos:**
1. Enviar POST com `"valor_litro": 50.00`

**Resultado Esperado:** HTTP 201.  
**Status:** ✅

---

## CT-011 — RN-01.3: `total_pago` enviado pelo cliente é ignorado

| Campo | Valor |
|-------|-------|
| **RN** | RN-01.3 |
| **Prioridade** | Alta |
| **Tipo** | Negócio |
| **Endpoint** | `POST /abastecimentos` |

**Passos:**
1. Enviar POST com `"total_pago": 9999.99` + campos válidos (`litros: 40.0`, `valor_litro: 5.99`)
2. Verificar `total_pago` na resposta

**Resultado Esperado:** HTTP 201; `total_pago` na resposta = `round(40.0 * 5.99, 2)` = `239.60`, não `9999.99`.  
**Status:** ❌

---

## CT-012 — RN-01.4: Arredondamento monetário com 2 casas decimais

| Campo | Valor |
|-------|-------|
| **RN** | RN-01.4 |
| **Prioridade** | Média |
| **Tipo** | Negócio |
| **Endpoint** | `POST /abastecimentos` |

**Passos:**
1. Enviar POST com `"litros": 33.333`, `"valor_litro": 3.0`
2. Verificar `total_pago` na resposta (deve ser `99.999` → arredondado para `100.00`)

**Resultado Esperado:** HTTP 201; `total_pago` = `100.00` (2 casas decimais).  
**Status:** ✅

---

## CT-013 — RN-02.1: `motorista_id` inexistente

| Campo | Valor |
|-------|-------|
| **RN** | RN-02.1 |
| **Prioridade** | Alta |
| **Tipo** | Negócio |
| **Endpoint** | `POST /abastecimentos` |

**Passos:**
1. Enviar POST com `"motorista_id": "mot_INVALIDO_9999"`

**Resultado Esperado:** HTTP 400 com mensagem indicando que o motorista não existe.  
**Status:** ✅

---

## CT-014 — RN-02.1: `veiculo_id` inexistente

| Campo | Valor |
|-------|-------|
| **RN** | RN-02.1 |
| **Prioridade** | Alta |
| **Tipo** | Negócio |
| **Endpoint** | `POST /abastecimentos` |

**Passos:**
1. Enviar POST com `"veiculo_id": "vei_INVALIDO_9999"`

**Resultado Esperado:** HTTP 400 com mensagem clara.  
**Status:** ✅

---

## CT-015 — RN-02.2: Deleção de registro reportado recentemente

| Campo | Valor |
|-------|-------|
| **RN** | RN-02.2 |
| **Prioridade** | Alta |
| **Tipo** | Negócio |
| **Endpoint** | `DELETE /abastecimentos/:id` |

**Pré-condição:** Registro criado e reportado via `POST /abastecimentos/:id/reportar-erro`.  
**Passos:**
1. Criar abastecimento (POST)
2. Reportar erro nele (POST `reportar-erro`)
3. Tentar deletar o abastecimento (DELETE)

**Resultado Esperado:** HTTP 409 (Conflict).  
**Status:** ❌

---

## CT-016 — RN-03.1: CPF mascarado na listagem

| Campo | Valor |
|-------|-------|
| **RN** | RN-03.1 |
| **Prioridade** | Alta |
| **Tipo** | Segurança |
| **Endpoint** | `GET /abastecimentos` |

**Passos:**
1. Enviar `GET /abastecimentos`
2. Inspecionar campo `motorista.cpf` nos itens retornados

**Resultado Esperado:** CPF no formato `XXX.***.***-XX` (com asteriscos).  
**Status:** ✅

---

## CT-017 — RN-03.1: CPF mascarado no detalhe

| Campo | Valor |
|-------|-------|
| **RN** | RN-03.1 |
| **Prioridade** | Alta |
| **Tipo** | Segurança |
| **Endpoint** | `GET /abastecimentos/:id` |

**Passos:**
1. Obter ID de abastecimento com motorista
2. Enviar `GET /abastecimentos/{id}`
3. Verificar `motorista.cpf`

**Resultado Esperado:** CPF mascarado (`XXX.***.***-XX`).  
**Status:** ❌

---

## CT-018 — RN-04.1: Filtro por `uf` case-insensitive

| Campo | Valor |
|-------|-------|
| **RN** | RN-04.1 |
| **Prioridade** | Média |
| **Tipo** | Funcional |
| **Endpoint** | `GET /abastecimentos?uf=` |

**Passos:**
1. Requisição com `uf=PE`
2. Requisição com `uf=pe`
3. Comparar `pagination.total`

**Resultado Esperado:** Totais iguais nas duas requisições.  
**Status:** ✅

---

## CT-019 — RN-04.2: Filtro por `combustivel` case-insensitive

| Campo | Valor |
|-------|-------|
| **RN** | RN-04.2 |
| **Prioridade** | Média |
| **Tipo** | Funcional |
| **Endpoint** | `GET /abastecimentos?combustivel=` |

**Passos:**
1. Requisição com `combustivel=GASOLINA`
2. Requisição com `combustivel=gasolina`
3. Comparar `pagination.total`

**Resultado Esperado:** Totais iguais.  
**Status:** ✅

---

## CT-020 — RN-04.3: `data_inicio` > `data_fim` retorna 400

| Campo | Valor |
|-------|-------|
| **RN** | RN-04.3 |
| **Prioridade** | Alta |
| **Tipo** | Negócio |
| **Endpoint** | `GET /abastecimentos` |

**Passos:**
1. Enviar GET com `data_inicio=2025-12-31&data_fim=2025-01-01`

**Resultado Esperado:** HTTP 400 com mensagem explicativa.  
**Status:** ❌

---

## CT-021 — RN-04.4: Paginação reflete filtros aplicados

| Campo | Valor |
|-------|-------|
| **RN** | RN-04.4 |
| **Prioridade** | Alta |
| **Tipo** | Negócio |
| **Endpoint** | `GET /abastecimentos` |

**Passos:**
1. GET sem filtro → registrar `pagination.total` (total_geral)
2. GET com `uf=PE` → registrar `pagination.total` (total_pe)
3. Verificar `total_pe <= total_geral`

**Resultado Esperado:** Total filtrado ≤ total geral; `total_pages` consistente com `total` e `limit`.  
**Status:** ✅

---

## CT-022 — RN-04.5: Página inexistente retorna 200 com lista vazia

| Campo | Valor |
|-------|-------|
| **RN** | RN-04.5 |
| **Prioridade** | Média |
| **Tipo** | Borda |
| **Endpoint** | `GET /abastecimentos?page=999999` |

**Passos:**
1. Enviar GET com `page=999999`

**Resultado Esperado:** HTTP 200 com `data: []`.  
**Status:** ✅

---

## CT-023 — RN-05.1: DELETE em ID inexistente retorna 404

| Campo | Valor |
|-------|-------|
| **RN** | RN-05.1 |
| **Prioridade** | Alta |
| **Tipo** | Negócio |
| **Endpoint** | `DELETE /abastecimentos/:id` |

**Passos:**
1. Enviar `DELETE /abastecimentos/abs_NAOEXISTE99999`

**Resultado Esperado:** HTTP 404 (não HTTP 500).  
**Status:** ✅

---

## CT-024 — RN-05.2: PUT em ID inexistente retorna 404

| Campo | Valor |
|-------|-------|
| **RN** | RN-05.2 |
| **Prioridade** | Alta |
| **Tipo** | Negócio |
| **Endpoint** | `PUT /abastecimentos/:id` |

**Passos:**
1. Enviar PUT com body válido para ID inexistente

**Resultado Esperado:** HTTP 404. Não deve criar o recurso (sem upsert).  
**Status:** ❌

---

## CT-025 — RN-05.3: GET de ID inexistente retorna 404 com payload correto

| Campo | Valor |
|-------|-------|
| **RN** | RN-05.3 |
| **Prioridade** | Alta |
| **Tipo** | Contrato |
| **Endpoint** | `GET /abastecimentos/:id` |

**Passos:**
1. Enviar `GET /abastecimentos/abs_NAOEXISTE99999`
2. Verificar status e body

**Resultado Esperado:** HTTP 404 com `{ "error": "...", "code": "NOT_FOUND" }`.  
**Status:** ✅

---

## CT-026 — RN-06.1: Múltiplos reportes geram protocolos únicos

| Campo | Valor |
|-------|-------|
| **RN** | RN-06.1 |
| **Prioridade** | Média |
| **Tipo** | Negócio |
| **Endpoint** | `POST /abastecimentos/:id/reportar-erro` |

**Passos:**
1. Criar abastecimento
2. Reportar erro duas vezes (mesma descrição)
3. Comparar `protocolo` das duas respostas

**Resultado Esperado:** HTTP 202 em ambos; protocolos diferentes.  
**Status:** ✅

---

## CT-027 — RN-06.2: `descricao` com menos de 10 caracteres rejeitada

| Campo | Valor |
|-------|-------|
| **RN** | RN-06.2 |
| **Prioridade** | Média |
| **Tipo** | Negócio |
| **Endpoint** | `POST /abastecimentos/:id/reportar-erro` |

**Passos:**
1. Criar abastecimento
2. Reportar com `"descricao": "curta"`

**Resultado Esperado:** HTTP 400.  
**Status:** ✅

---

## CT-028 — RN-06.2: `descricao` com mais de 500 caracteres rejeitada

| Campo | Valor |
|-------|-------|
| **RN** | RN-06.2 |
| **Prioridade** | Média |
| **Tipo** | Borda |
| **Endpoint** | `POST /abastecimentos/:id/reportar-erro` |

**Passos:**
1. Criar abastecimento
2. Reportar com `descricao` de 501 caracteres

**Resultado Esperado:** HTTP 400.  
**Status:** ✅

---

## CT-029 — RN-06.3: Reporte não modifica o abastecimento original

| Campo | Valor |
|-------|-------|
| **RN** | RN-06.3 |
| **Prioridade** | Alta |
| **Tipo** | Negócio |
| **Endpoint** | `POST /abastecimentos/:id/reportar-erro` |

**Passos:**
1. Criar abastecimento, registrar campos principais
2. Reportar erro
3. `GET /abastecimentos/{id}` e comparar campos

**Resultado Esperado:** Campos `litros`, `valor_litro`, `total_pago`, `combustivel`, `uf` inalterados.  
**Status:** ✅

---

## CT-030 — RN-07.1: Data futura (> 1h) rejeitada

| Campo | Valor |
|-------|-------|
| **RN** | RN-07.1 |
| **Prioridade** | Alta |
| **Tipo** | Negócio |
| **Endpoint** | `POST /abastecimentos` |

**Passos:**
1. Enviar POST com `data` = agora + 2 horas (ISO 8601)

**Resultado Esperado:** HTTP 400.  
**Status:** ❌

---

## CT-031 — RN-07.1: Data com 30 min no futuro aceita (dentro da tolerância)

| Campo | Valor |
|-------|-------|
| **RN** | RN-07.1 |
| **Prioridade** | Média |
| **Tipo** | Borda |
| **Endpoint** | `POST /abastecimentos` |

**Passos:**
1. Enviar POST com `data` = agora + 30 minutos

**Resultado Esperado:** HTTP 201.  
**Status:** ✅

---

## CT-032 — RN-07.2: Data anterior a 01/01/2020 rejeitada

| Campo | Valor |
|-------|-------|
| **RN** | RN-07.2 |
| **Prioridade** | Alta |
| **Tipo** | Negócio |
| **Endpoint** | `POST /abastecimentos` |

**Passos:**
1. Enviar POST com `"data": "2019-12-31T23:59:59Z"`

**Resultado Esperado:** HTTP 400.  
**Status:** ❌


---

## CT-033 — RN-07.2: Data exatamente em 01/01/2020 aceita

| Campo | Valor |
|-------|-------|
| **RN** | RN-07.2 |
| **Prioridade** | Média |
| **Tipo** | Borda |
| **Endpoint** | `POST /abastecimentos` |

**Passos:**
1. Enviar POST com `"data": "2020-01-01T00:00:00Z"`

**Resultado Esperado:** HTTP 201.  
**Status:** ✅

---

## CT-034 — RN-08.1: Veículo elétrico com combustível convencional rejeitado

| Campo | Valor |
|-------|-------|
| **RN** | RN-08.1 |
| **Prioridade** | Média |
| **Tipo** | Negócio |
| **Endpoint** | `POST /abastecimentos` |

**Pré-condição:** Existência de um veículo cadastrado com `tipo: "ELETRICO"`.  
**Passos:**
1. Identificar `veiculo_id` de veículo elétrico
2. Enviar POST com `"combustivel": "GASOLINA"` e esse `veiculo_id`

**Resultado Esperado:** HTTP 400.  
**Observação:** Se não houver veículo elétrico cadastrado, documentar limitação e marcar como bloqueado.  
**Status:** ⏭️

---

## CT-035 a CT-043 — Campos obrigatórios ausentes (parametrizado)

| CT | Campo ausente | RN | Resultado Esperado |
|----|---------------|----|--------------------|
| CT-035 | `data` | — | HTTP 400 |
| CT-036 | `posto` | — | HTTP 400 |
| CT-037 | `cidade` | — | HTTP 400 |
| CT-038 | `uf` | — | HTTP 400 |
| CT-039 | `combustivel` | — | HTTP 400 |
| CT-040 | `litros` | — | HTTP 400 |
| CT-041 | `valor_litro` | — | HTTP 400 |
| CT-042 | `motorista_id` | — | HTTP 400 |
| CT-043 | `veiculo_id` | — | HTTP 400 |

**Status geral:** ✅

---

## CT-044 — Combinação de filtros: UF + combustível + período

| Campo | Valor |
|-------|-------|
| **RN** | RN-04 |
| **Prioridade** | Média |
| **Tipo** | Funcional |
| **Endpoint** | `GET /abastecimentos` |

**Passos:**
1. GET com `uf=SP&combustivel=DIESEL&data_inicio=2025-01-01&data_fim=2025-12-31`

**Resultado Esperado:** HTTP 200 com dados filtrados consistentes.  
**Status:** ✅

---

## Resumo de Cobertura por RN

| Regra de Negócio | CTs Cobertos | Total de CTs |
|------------------|:---:|:---:|
| CRUD Básico | CT-001 a CT-005 | 5 |
| RN-01 (Valores Monetários) | CT-006 a CT-012 | 7 |
| RN-02 (Integridade Referencial) | CT-013 a CT-015 | 3 |
| RN-03 (LGPD/CPF) | CT-016 a CT-017 | 2 |
| RN-04 (Filtros/Paginação) | CT-018 a CT-022, CT-044 | 6 |
| RN-05 (Idempotência) | CT-023 a CT-025 | 3 |
| RN-06 (Reportar Erro) | CT-026 a CT-029 | 4 |
| RN-07 (Consistência Temporal) | CT-030 a CT-033 | 4 |
| RN-08 (Combustível/Veículo) | CT-034 | 1 |
| Campos Obrigatórios | CT-035 a CT-043 | 9 |
| **Total** | | **44** |

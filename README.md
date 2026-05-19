# Desafio QA V-Lab — API de Combustíveis

## Resumo Executivo

A API de Combustíveis (`https://v-lab-testes.onrender.com/v1`) foi submetida a uma execução de **44 casos de teste manuais via Postman**, cobrindo o ciclo CRUD completo, todas as regras de negócio (RN-01 a RN-08), validações de campo, filtros, paginação e privacidade de dados. **33 testes passaram, 10 revelaram defeitos reais confirmados e 1 permanece bloqueado por impedimento de infraestrutura**. Entre os defeitos, destacam-se 2 de severidade **Crítica**: (1) o campo `total_pago` enviado pelo cliente é persistido diretamente pela API — permitindo a manipulação arbitrária de valores financeiros — e (2) o CPF do motorista é retornado sem mascaramento no endpoint de detalhe (`GET /abastecimentos/:id`), em violação direta à LGPD. Adicionalmente, a API não recalcula `total_pago` em atualizações via PUT, não rejeita valores de litro fora dos limites permitidos, permite a deleção de registos sob reporte ativo, não valida intervalos de data nos filtros e realiza upsert silencioso em IDs inexistentes. A API **não está apta para produção** no estado atual.

---

## Tabela de Bugs

| ID do CT | Título | Severidade | Status |
|----------|--------|:---:|:---:|
| CT-004 | `total_pago` não é recalculado no PUT | Alta | Aberto |
| CT-008 | `valor_litro = 0` aceito | Alta | Aberto |
| CT-009 | `valor_litro > R$50,00` aceito sem rejeição | Alta | Aberto |
| CT-011 | `total_pago` enviado pelo cliente é persistido | **Crítica** | Aberto |
| CT-015 | Deleção de registo reportado retornou 204 | Alta | Aberto |
| CT-017 | CPF exposto sem máscara no detalhe do abastecimento | **Crítica** | Aberto |
| CT-020 | Busca com `data_inicio > data_fim` retornou 200 | Média | Aberto |
| CT-024 | PUT com ID inexistente realizou upsert silencioso | Alta | Aberto |
| CT-030 | API aceitou data superior a 1h no futuro | Alta | Aberto |
| CT-032 | API aceitou data anterior a 01/01/2020 | Alta | Aberto |

---

## Como Reproduzir os Testes

Todos os testes foram executados e documentados utilizando o **Postman**. A Collection exportada (`API_Combustiveis_VLab_Postman.json`) contém todas as requests organizadas por regra de negócio, com os payloads utilizados e os **Examples** guardados diretamente em cada request (resposta real da API no momento da execução).

### Pré-requisitos

- [Postman](https://www.postman.com/downloads/) instalado (versão Desktop recomendada)
- Acesso à internet (a API está hospedada em `https://v-lab-testes.onrender.com`)

### Importar a Collection no Postman

1. Abre o Postman.
2. Clica em **Import** (canto superior esquerdo).
3. Seleciona o ficheiro `API_Combustiveis_VLab_Postman.json` presente na raiz deste repositório.
4. A Collection **"API Combustíveis V-Lab"** aparecerá na barra lateral esquerda, organizada em pastas por regra de negócio (ex: `RN-01 Validações`, `RN-02 Auditoria`, `RN-03 LGPD`, etc.).

### Ver as Requests e os Examples guardados

1. Expande qualquer pasta na barra lateral para ver as requests individuais.
2. Clica numa request (ex: `CT-011 - total_pago enviado pelo cliente`).
3. No painel central, clica no separador **Examples** (abaixo do nome da request).
4. O Example guardado mostra o payload enviado e a resposta real obtida da API — incluindo o status HTTP e o corpo da resposta — servindo como evidência directa do comportamento observado.

### Reproduzir um bug específico

Para reproduzir qualquer bug manualmente, basta abrir a request correspondente ao CT e clicar em **Send**. O comportamento defeituoso será imediatamente visível na resposta. Os cURLs de cada bug estão documentados individualmente no `RELATORIO_DE_BUGS.md`.

---

## Estrutura do Repositório

```
/
├── README.md                               # Este ficheiro
├── PLANO_DE_TESTE.md                       # Escopo, estratégia e gestão de riscos
├── CASOS_DE_TESTE.md                       # 44 casos de teste documentados
├── RELATORIO_DE_BUGS.md                    # 10 bugs confirmados com evidências (cURLs)
└── API_Combustiveis_VLab_Postman.json      # Collection Postman exportada com Examples
```

---

## Distribuição de Tempo

| Etapa | Atividade | Tempo |
|-------|-----------|:---:|
| 1 | Leitura e análise da documentação, requisitos e critérios de aceitação | 1h |
| 2 | Exploração manual da API (smoke test, descoberta de IDs e comportamentos) | 1h |
| 3 | Planeamento e escrita dos Casos de Teste (`CASOS_DE_TESTE.md`) | 3h |
| 4 | Criação da Collection no Postman e execução dos 44 testes | 3h |
| 5 | Análise de falhas, escrita do Relatório de Bugs e documentação de evidências | 2h |
| 6 | Organização do repositório e revisão final | 1h |
| **Total** | | **~11h** |

---

## O que Ficou de Fora e Por Quê

| Item | Justificativa |
|------|---------------|
| Testes de carga / stress | O rate limit da API (15 req/min) inviabiliza testes de volume com ferramentas como k6 ou Locust; adicionalmente, testes de carga estavam fora do escopo explícito do desafio |
| Script de automação (pytest / Playwright) | A estratégia foi deliberadamente orientada para a **profundidade analítica**: boundary testing rigoroso, validação manual de todas as regras de negócio e documentação detalhada de cada comportamento observado. A execução manual via Postman permite inspecionar e documentar cada resposta individualmente, com maior fidelidade às evidências |
| Testes de segurança (SQLi, XSS, fuzzing) | Fora do escopo explícito do desafio |
| CT-034 (Veículo Elétrico vs. Gasolina) | Bloqueado por impedimento de infraestrutura: a API não disponibiliza endpoint de cadastro de veículos (`POST /v1/veiculos`), impossibilitando a criação do pré-requisito de dados necessário para validar a RN-08.1. Detalhe completo em `RELATORIO_DE_BUGS.md` |

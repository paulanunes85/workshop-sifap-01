<!-- markdownlint-disable MD012 MD013 MD022 MD025 MD026 MD028 MD029 MD031 MD033 MD034 MD038 MD040 MD051 MD060 -->

# Catálogo de Regras de Negócio — SIFAP Legado

![ESTÁGIO 01 Arqueologia](https://img.shields.io/badge/ESTÁGIO-01%20Arqueologia-F25022?style=for-the-badge) ![TIPO Worksheet](https://img.shields.io/badge/TIPO-Worksheet-1A1A1A?style=for-the-badge) ![PREENCHA Durante S1](https://img.shields.io/badge/PREENCHA-Durante%20S1-737373?style=for-the-badge)

> 🗺 **Você está aqui:** [Kit PT-BR](../README.md) → [Estágio 1](README.md) → **business-rules-catalog**

> **Estágio 1 · `/extract-business-rules`**
>
> Cada par extrai as regras dos programas `.NSN` atribuídos a ele e registra aqui.
> Cada regra cita o programa-fonte com faixa de linhas (`arquivo.NSN:Lstart-Lend`) e é
> classificada como **Confirmada** (cruza com a documentação histórica em
> `legado-sifap/legacy-docs/`), **Inferida** (somente do código) ou
> **Mistério** (pergunta em aberto → registre também em
> [`mysteries-found.md`](mysteries-found.md) com evidência `path:linha`, hipótese não
> confirmada, responsável e status).
>
> 📘 **Guia passo a passo:** [`GUIDE.md`](GUIDE.md).

**Time**: <!-- preencher -->

---

## Regras de `VALELEG.NSN`

**Programa:** `01-arqueologia/legado-sifap/natural-programs/VALELEG.NSN` (244 linhas)
**Objetivo declarado no cabeçalho:** validação de elegibilidade de beneficiário para programa (arq. 150/155)
**Histórico de alterações no cabeçalho:** 1999 criação · 2004 novas regras eleg. · 2009 ajuste faixa etária · 2013 inclusão região 99
**Documentação cruzada:** `legacy-docs/REGRAS-NEGOCIO-2012.md` §1, §4.1, §4.2
**DDM cruzado:** `adabas-ddms/BENEFICIARIO.ddm` (FNR 150)

### Guardas de entrada (abortam a execução)

| #   | Declaração da Regra | Candidata EARS | Fonte | Classificação | Observações |
| --- | --- | --- | --- | --- | --- |
| 1 | Se o CPF informado não existir no cadastro de beneficiários, então o sistema deve exibir "BENEFICIARIO NAO ENCONTRADO" e encerrar sem avaliar elegibilidade. | Unwanted | `VALELEG.NSN:L69-84` | Inferida | Sem suporte documental. `FIND` sem `IF NO RECORDS FOUND`; usa flag manual `#FOUND-B`. |
| 2 | Se o código de programa informado não existir, então o sistema deve exibir "PROGRAMA NAO ENCONTRADO" e encerrar. | Unwanted | `VALELEG.NSN:L87-97` | Inferida | Mesmo padrão de flag manual (`#FOUND-P`). |
| 3 | Se o programa social não estiver com status `'A'` (ativo), então o sistema deve exibir "PROGRAMA INATIVO" e encerrar. | Unwanted | `VALELEG.NSN:L99-102` | Inferida | Parcialmente alinhado a RN-003, que fala em `PS-IN-ATIVO = 'S'` — **campo e valor diferentes do código**. |
| 4 | Onde o código de região do beneficiário for `99`, o sistema deve declará-lo elegível e encerrar, **sem executar nenhuma das demais validações**. | Optional | `VALELEG.NSN:L107-111` | **Mistério** | 🚨 Bypass total. Ver M-01. <!-- mystery: bypass integral de elegibilidade por COD-REGIAO=99; finalidade real desconhecida; doc de 2012 registra a existência mas nenhum autor soube explicar --> |

### Situação cadastral do beneficiário

| #   | Declaração da Regra | Candidata EARS | Fonte | Classificação | Observações |
| --- | --- | --- | --- | --- | --- |
| 5 | Se a situação do beneficiário for `'S'`, então o sistema deve marcá-lo inelegível com o motivo "BENEFICIARIO SUSPENSO". | Unwanted | `VALELEG.NSN:L116-121` | **Confirmada** | REGRAS-NEGOCIO-2012 §4.2: "situação cadastral ativa (`BN-CD-SIT = 'A'`)". DDM: `S=SUSP`. |
| 6 | Se a situação do beneficiário for `'C'` ou `'D'`, então o sistema deve marcá-lo inelegível com o motivo "BENEFICIARIO CANCELADO/DESLIGADO". | Unwanted | `VALELEG.NSN:L122-126` | **Confirmada** | DDM: `C=CANC`, `D=DESL`. |
| 7 | Se a situação do beneficiário for `'I'`, então o sistema deve marcá-lo inelegível com o motivo "BENEFICIARIO INATIVO". | Unwanted | `VALELEG.NSN:L127-131` | **Confirmada** | DDM: `I=INAT`. |

> ⚠️ **Divergência de domínio.** RN-011 afirma que a exclusão é lógica via `BN-CD-SIT = 'E'`.
> O valor `'E'` **não existe** no DDM (`A/S/C/I/D`) nem é tratado no código. Um beneficiário
> com situação `'E'` cairia no `IF #STATUS-BENEF NE 'A'` sem casar com nenhum `ELSE`
> aninhado — ficaria **inelegível silenciosamente, sem motivo registrado** (`VALELEG.NSN:L116-134`).
> <!-- mystery: valor de situação 'E' citado em RN-011 não existe no DDM nem no código; qualquer valor fora de A/S/C/I/D produz reprovação sem motivo -->

### Faixa etária e renda (parametrizadas por programa)

| #   | Declaração da Regra | Candidata EARS | Fonte | Classificação | Observações |
| --- | --- | --- | --- | --- | --- |
| 8 | Onde o programa definir idade mínima maior que zero, se a idade do beneficiário for inferior a ela, então o sistema deve marcá-lo inelegível. | Unwanted | `VALELEG.NSN:L139-145` | Inferida | `IDADE-MIN = 0` funciona como "sem restrição" — sentinela não documentada. |
| 9 | Onde o programa definir idade máxima maior que zero, se a idade do beneficiário for superior a ela, então o sistema deve marcá-lo inelegível. | Unwanted | `VALELEG.NSN:L146-152` | Inferida | Mesma sentinela `0`. |
| 10 | Onde o programa definir renda máxima maior que zero, se a **renda familiar total** do beneficiário for superior a ela, então o sistema deve marcá-lo inelegível. | Unwanted | `VALELEG.NSN:L157-163` | **Mistério** | Doc §4.2 e RN-018 falam em **renda per capita**; o código usa renda familiar **total**. O DDM tem os dois campos (`CH VLR-RENDA-FAMILIAR`, `CJ IND-RENDA-PERCAP`) e o per capita **nunca é lido**. Ver M-02. |

### Regras por tipo de programa (`DECIDE ON FIRST VALUE`)

| #   | Declaração da Regra | Candidata EARS | Fonte | Classificação | Observações |
| --- | --- | --- | --- | --- | --- |
| 11 | Onde o programa for do tipo `'A'` (assistencial), se a renda familiar for superior a `600,00` **e** o beneficiário não tiver nenhum dependente, então o sistema deve marcá-lo inelegível. | Unwanted | `VALELEG.NSN:L169-177` | **Mistério** | 🚨 Magic number `600.00` em hard-code, sem suporte documental e sem correção monetária desde 1999. Note a condição **aninhada (E lógico)**: renda alta **com** dependente passa. Ver M-03. |
| 12 | Onde o programa for do tipo `'A'`, se o indicador de documentação não for `'S'`, então o sistema deve marcá-lo inelegível com o motivo "DOCUMENTACAO INCOMPLETA". | Unwanted | `VALELEG.NSN:L178-182` | **Mistério** | O campo `DOCUMENTOS-OK` **não existe no DDM BENEFICIARIO**. Ver M-04. |
| 13 | Onde o programa for do tipo `'P'` (previdenciário), se a idade for inferior a 60 anos, então o sistema deve marcá-lo inelegível. | Unwanted | `VALELEG.NSN:L183-189` | Inferida | Idade 60 em hard-code, duplicando o mecanismo parametrizado de `IDADE-MIN` (regra 8). Redundância não documentada. |
| 14 | Onde o programa for do tipo `'T'` (trabalho), se a idade estiver fora da faixa de 16 a 65 anos, então o sistema deve marcá-lo inelegível. | Unwanted | `VALELEG.NSN:L190-196` | **Confirmada (parcial)** | Limite inferior 16 alinha com RN-006 ("idade inferior a 16 anos" vedada). Limite superior 65 **não tem suporte documental**. |
| 15 | Se o tipo do programa não for `'A'`, `'P'` nem `'T'`, então o sistema deve marcar o beneficiário inelegível com o motivo "TIPO PROGRAMA DESCONHECIDO". | Unwanted | `VALELEG.NSN:L197-200` | Inferida | Branch `NONE` — fail-closed. Confirmar valores válidos de `TIPO` em `PROGRAMA-SOCIAL.ddm`. |

### Elegibilidade específica por código (`COD-ELEGIBILIDADE`, A5 posicional)

| #   | Declaração da Regra | Candidata EARS | Fonte | Classificação | Observações |
| --- | --- | --- | --- | --- | --- |
| 16 | Onde a 1ª posição do código de elegibilidade for `'R'`, se o NIS do beneficiário for zero, então o sistema deve marcá-lo inelegível com o motivo "NIS NAO CADASTRADO". | Unwanted | `VALELEG.NSN:L226-233` | **Mistério** | RN-001 exige "NIS/NIT **ativo**, validado pelo subprograma `VALNISN`". O código apenas testa `≠ 0`, e **`VALNISN` não existe no inventário** dos 15 programas. Ver M-05. |
| 17 | Onde a 2ª posição do código de elegibilidade for `'D'`, se o beneficiário não tiver dependentes, então o sistema deve marcá-lo inelegível. | Unwanted | `VALELEG.NSN:L234-241` | **Mistério** | Conflito triplo sobre o limite de dependentes. Ver M-06. |
| — | As posições 3, 4 e 5 do código `A5` **nunca são lidas**. | — | `VALELEG.NSN:L32, L226-241` | **Mistério** | <!-- mystery: COD-ELEGIBILIDADE tem 5 posições mas só as posições 1 e 2 são interpretadas; desconhecido se 3-5 são reservadas, obsoletas ou tratadas em outro programa --> |

### Comportamento estrutural

| #   | Declaração da Regra | Candidata EARS | Fonte | Classificação | Observações |
| --- | --- | --- | --- | --- | --- |
| 18 | O sistema deve calcular a idade como a diferença entre o ano corrente e o ano de nascimento. | Ubiquitous | `VALELEG.NSN:L59, L72-73` | Inferida | ⚠️ **Defeito latente:** ignora mês e dia. Quem faz aniversário depois da data corrente é contado com 1 ano a mais. Impacta diretamente os cortes 16 / 60 / 65 (regras 8, 9, 13, 14). |
| 19 | O sistema deve avaliar **todas** as regras aplicáveis e acumular todos os motivos de recusa, em vez de parar na primeira falha. | Ubiquitous | `VALELEG.NSN:L41, L118-241` | Inferida | `#MOTIVO (A60/10)` — máximo teórico de motivos acumuláveis é 8, então não há estouro do array. |
| 20 | Ao final, se o beneficiário permanecer elegível o sistema deve informar a aprovação; caso contrário deve listar todos os motivos numerados. | Ubiquitous | `VALELEG.NSN:L213-220` | Inferida | Saída via `WRITE` (terminal). Nenhum resultado é persistido nem auditado — ver M-07. |

### Regras documentadas que **não foram encontradas** no código

O `REGRAS-NEGOCIO-2012.md` §4.2 afirma que estas regras estão em `VALELEG`. Após leitura completa das 244 linhas, **nenhuma delas existe no programa**:

| ID | Regra documentada | Fonte da alegação | Situação no código |
| --- | --- | --- | --- |
| M-08 | "Beneficiário deve possuir dados bancários válidos e completos" | §4.2 · RN-007 | ❌ Ausente. Nenhum campo bancário sequer é declarado na VIEW. |
| M-09 | "Não pode estar inscrito em mais de 2 programas simultâneos (`BN-QT-PROG`)" | §4.2 | ❌ Ausente. O campo **não existe no DDM**. |
| M-10 | "Data de última atualização cadastral não pode exceder 24 meses (`BN-DT-ULT-ATUAL`)" | §4.2 | ❌ Ausente. O DDM tem `GD DT-ULT-ALTERACAO`, nunca lida aqui. |
| M-11 | "Não pode ter ocorrência de auditoria não resolvida do tipo `'B'` (bloqueio)" | §4.2 | ❌ Ausente. `AUDITORIA.ddm` não é referenciado por este programa. |

<!-- mystery: 4 regras de elegibilidade descritas na documentação de 2012 não têm nenhuma implementação em VALELEG.NSN; desconhecido se foram removidas, se nunca existiram, ou se residem em outro programa não identificado -->

> 📏 **Sinal de alerta adicional:** o documento de 2012 afirma que "o programa possui
> aproximadamente 1.200 linhas". O arquivo tem **244 linhas**. Ou o documento descreve
> outra versão, ou a análise de 2012 se baseou em código que não é este.
> <!-- mystery: divergência de tamanho entre o VALELEG descrito na doc de 2012 (~1200 linhas) e o arquivo disponível (244 linhas) -->

### Mistérios abertos (resumo para `/catalog-mysteries`)

| ID | Pergunta em aberto | Evidência | Impacto |
| --- | --- | --- | --- |
| M-01 | Qual a finalidade real do bypass por região `99`, e por que ele precede a checagem de situação cadastral? | `VALELEG.NSN:L107-111`; `REGRAS-NEGOCIO-2012.md` §1 nota RN-005 e §4.2 nota final | 🔴 Crítico. Beneficiário **suspenso ou cancelado** na região 99 é aprovado. Comentário do código diz "INTERNACIONAL/DIPLOMATICO"; o cabeçalho data a inclusão em 2013; a doc de 2012 **já registrava a existência** do desvio. As três fontes se contradizem. |
| M-02 | A regra de renda deve usar renda familiar total (código) ou renda per capita (documentação)? | `VALELEG.NSN:L157-163`; `BENEFICIARIO.ddm` campos `CH` e `CJ`; RN-018 | 🔴 Crítico. Muda quem é elegível. |
| M-03 | De onde vem o limite `600,00` e por que ele só reprova quem **não** tem dependentes? | `VALELEG.NSN:L171-177` | 🟠 Alto. Valor congelado, sem indexação, sem doc. |
| M-04 | Onde é armazenado `DOCUMENTOS-OK`? | `VALELEG.NSN:L25, L178`; ausente em `BENEFICIARIO.ddm` | 🟠 Alto. Campo lido mas inexistente no DDM. |
| M-05 | Validar NIS é "diferente de zero" (código) ou "NIS ativo via `VALNISN`" (RN-001)? Onde está `VALNISN`? | `VALELEG.NSN:L228`; RN-001; `inventory.md` | 🟠 Alto. Subprograma citado não está entre os 15 arquivos. |
| M-06 | Qual é o limite real de dependentes: **3** (RN-004), **5** (nota da RN-004) ou **10** (grupo `PE` do DDM)? | RN-004 + nota; `BENEFICIARIO.ddm` `1 DA GRP-DEPENDENTE PE ... 10` | 🟠 Alto. Define a cardinalidade no schema-alvo. |
| M-07 | O resultado da validação é persistido ou auditado em algum lugar? | `VALELEG.NSN:L213-220` | 🟡 Médio. RN-010 exige auditoria de alterações; aqui não há gravação. |
| M-12 | A VIEW do programa **não bate com o DDM**: nomes e tipos divergem. | Ver tabela abaixo | 🔴 Crítico. Bloqueia a modelagem do PostgreSQL. |

#### M-12 em detalhe — VIEW × DDM

| Campo na VIEW (`VALELEG.NSN:L15-25`) | Campo correspondente no DDM | Divergência |
| --- | --- | --- |
| `CPF (N11)` | `AB NUM-CPF (A11)` | Nome **e** tipo (numérico × alfanumérico) |
| `STATUS (A1)` | `CE SIT-BENEFICIARIO (A1)` | Nome |
| `COD-PROGRAMA (N4)` | `CA COD-PROGRAMA (A4)` | Tipo (N × A) |
| `RENDA-FAMILIAR (N9.2)` | `CH VLR-RENDA-FAMILIAR (N9.2)` | Nome |
| `COD-REGIAO (N2)` | `BJ COD-REGIAO (A2)` | Tipo — **e o código compara com o literal numérico `99`** |
| `NUM-DEPENDENTES (N2)` | ❌ inexistente (há `CI QTD-MEMBROS-FAMILIA` e o grupo `PE`) | Conceito diferente: membros do domicílio ≠ dependentes |
| `NIS (N11)` | ❌ inexistente (talvez `AA NUM-INSCRICAO`?) | Não confirmado |
| `DOCUMENTOS-OK (A1)` | ❌ inexistente | Sem origem conhecida |

> Some-se a isso um **quarto vocabulário**: a documentação usa prefixos `BN-*` (`BN-CD-SIT`,
> `BN-CD-REGIAO`), que não aparecem nem no DDM nem no programa. E o intervalo válido de
> região tem três versões: RN-005 diz `01–27`, o DDM diz `01–05 ou 99`, e o código só testa `99`.

### Rascunhos EARS (para o Estágio 2)

Somente para as regras **Confirmadas**. Cada uma carrega `source_legacy:` conforme exigido pelo CI `legacy-traceability`.

```markdown
REQ-001 — Se a situação cadastral do beneficiário for diferente de "ativa",
então o sistema deverá recusar a elegibilidade e registrar o motivo correspondente.
source_legacy: 01-arqueologia/legado-sifap/natural-programs/VALELEG.NSN:L116-134

REQ-002 — Se o beneficiário estiver com situação "suspensa",
então o sistema deverá recusar a elegibilidade com o motivo "beneficiário suspenso".
source_legacy: 01-arqueologia/legado-sifap/natural-programs/VALELEG.NSN:L117-121

REQ-003 — Se o beneficiário estiver com situação "cancelada" ou "desligada",
então o sistema deverá recusar a elegibilidade com o motivo correspondente.
source_legacy: 01-arqueologia/legado-sifap/natural-programs/VALELEG.NSN:L122-126

REQ-004 — Onde o programa social for do tipo "trabalho",
se a idade do beneficiário for inferior a 16 anos,
então o sistema deverá recusar a elegibilidade.
source_legacy: 01-arqueologia/legado-sifap/natural-programs/VALELEG.NSN:L190-196

REQ-005 — Quando uma validação de elegibilidade for concluída com recusa,
o sistema deverá apresentar a lista completa de motivos, e não apenas o primeiro.
source_legacy: 01-arqueologia/legado-sifap/natural-programs/VALELEG.NSN:L213-220
```

> ⚠️ **Não gere EARS a partir das regras classificadas como Mistério.** Elas precisam de
> validação humana primeiro. Leve M-01, M-02 e M-12 para a Passagem #1 como decisões
> bloqueantes de escopo.

---

# 🚨 Achados estruturais (valem para os 15 programas)

## E-01 · Não existe uma única chamada entre programas

Busca por `CALLNAT`, `INCLUDE` e `FETCH` nos 15 arquivos `.NSN`: **zero ocorrências**.
As 23 ocorrências encontradas são todas `PERFORM` — sub-rotinas **internas** ao próprio arquivo.

**Consequências diretas:**

- O call graph entre programas tem **zero arestas**. Os 15 programas são ilhas isoladas.
- Todo o acoplamento acontece **através dos arquivos Adabas compartilhados** (150/151/152/153).
- Os subprogramas citados na documentação — `VALCPF` (RN-001), `VALNISN` (RN-001), `LOGAUDIT` (RN-010), `CALCIDX` (RN-019) — **não existem e não poderiam ser chamados**.
- `VALBENEF` e `VALDOCS` têm nome de rotina de validação reutilizável, mas **ninguém os chama**. São executáveis isolados.

<!-- mystery: nenhum programa chama outro; desconhecido se o SIFAP real usa CALLNAT e o kit removeu, ou se a arquitetura sempre foi de programas isolados acoplados por arquivo -->

## E-02 · O algoritmo de CPF está triplicado com comportamentos divergentes

| Programa | Sub-rotina | Checa dígitos repetidos? | Backdoor? | Fonte |
| --- | --- | --- | --- | --- |
| `CADBENEF` | `VALIDA-CPF` | ❌ Não | — | `CADBENEF.NSN:L224-269` |
| `VALBENEF` | `VALIDA-CPF-COMPLETO` | ✅ Sim | 🚨 CPF `000...0` aceito | `VALBENEF.NSN:L185-199` |
| `VALDOCS` | `VALIDA-CPF-DOC` | ❌ Não | 🚨 8 prefixos ignoram tudo | `VALDOCS.NSN:L49-56, L174` |

**O mesmo CPF pode ser válido em um programa e inválido em outro.** Não há fonte única da verdade.

## E-03 · Nenhum programa grava auditoria

RN-010 exige registro automático em `AUDITORIA` (FNR 153) para toda alteração cadastral.
`CADBENEF` e `CADDEPEND` fazem `STORE`/`UPDATE` em produção **sem escrever uma linha de auditoria**.
O único programa que toca `AUDITORIA.ddm` é `RELAUDIT` — e apenas para leitura.

<!-- mystery: obrigatoriedade legal IN-TCU 63/2010 declarada no DDM AUDITORIA, mas nenhum programa de manutenção grava trilha; desconhecido se existe gravação por trigger Adabas ou se a exigência nunca foi implementada -->

---

## Regras de `CADBENEF.NSN`

**Programa:** `01-arqueologia/legado-sifap/natural-programs/CADBENEF.NSN` (271 linhas) · Par 1 · Visão
**Cabeçalho:** 1997 criação (Carlos Roberto da Silva) · 2005 validação de CPF · **2011 "AJUSTE STATUS IDOSO"**
**Acesso a dados:** lê e escreve `BENEFICIARIO` (FNR 150) — `STORE` e `UPDATE`

| #   | Declaração da Regra | Candidata EARS | Fonte | Classificação | Observações |
| --- | --- | --- | --- | --- | --- |
| 21 | Se a operação informada não for `'I'` (inclusão) nem `'A'` (alteração), então o sistema deve recusar. | Unwanted | `CADBENEF.NSN:L99-103` | Inferida | Usa `ESCAPE BOTTOM` fora de qualquer laço — ver nota de sintaxe abaixo. |
| 22 | Se o CPF não for informado, então o sistema deve recusar com "CPF OBRIGATORIO". | Unwanted | `CADBENEF.NSN:L105-109` | **Confirmada** | RN-001: "todo beneficiário deve possuir CPF válido". |
| 23 | Se o CPF não passar na validação de dígito verificador (módulo 11), então o sistema deve recusar. | Unwanted | `CADBENEF.NSN:L112-117, L224-269` | **Confirmada** | RN-001 cita o subprograma `VALCPF` — que **não existe**; a lógica está embutida. |
| 24 | Se o nome não for informado, então o sistema deve recusar. | Unwanted | `CADBENEF.NSN:L119-123` | Inferida | Só testa vazio. Não exige sobrenome — ao contrário de `VALBENEF` (regra 44). |
| 25 | Se a data de nascimento não for informada, então o sistema deve recusar. | Unwanted | `CADBENEF.NSN:L125-129` | **Confirmada** | RN-006: "data de nascimento é campo obrigatório". |
| 26 | Se o sexo não for `'M'` nem `'F'`, então o sistema deve recusar. | Unwanted | `CADBENEF.NSN:L131-135` | **Mistério** | 🚨 O DDM aceita `M/F/I` (`AG SEXO`, `I=INDEFINIDO`). O código **rejeita `'I'`**. Ver M-13. |
| 27 | Se a operação for inclusão e o CPF já existir, então o sistema deve recusar. | Unwanted | `CADBENEF.NSN:L143-147` | **Confirmada (parcial)** | RN-002 restringe a duplicidade a beneficiários **em situação ativa**; o código bloqueia **qualquer** duplicidade, impedindo a reinclusão que a RN-002 prevê. |
| 28 | Se a operação for alteração e o CPF não existir, então o sistema deve recusar. | Unwanted | `CADBENEF.NSN:L149-153` | Inferida | — |
| 29 | Quando um beneficiário for incluído, o sistema deve atribuir situação `'A'` (ativa). | Event-driven | `CADBENEF.NSN:L162-164` | **Confirmada** | RN-002 e §4.2. |
| 30 | **Se a idade do beneficiário for superior a 75 anos, então o sistema deve atribuir situação `'S'` (suspenso).** | Unwanted | `CADBENEF.NSN:L166-169` | **Mistério** | 🚨🚨 Ver M-14. O bloco fica **fora** do `IF #OPER = 'I'`, então dispara também em **toda alteração**. Combinado com a regra 5, todo beneficiário com mais de 75 anos torna-se **permanentemente inelegível**. |
| 31 | Quando a operação for inclusão, o sistema deve gravar o registro e registrar data de cadastro e de atualização com a data corrente. | Event-driven | `CADBENEF.NSN:L178-199` | Inferida | `STORE` + `END TRANSACTION`. Sem auditoria (E-03). |
| 32 | Quando a operação for alteração, o sistema deve atualizar apenas os campos editáveis e a data de atualização. | Event-driven | `CADBENEF.NSN:L200-216` | Inferida | ⚠️ **Não atualiza** `COD-PROGRAMA`, `COD-REGIAO` nem `NIS` — silenciosamente, sem avisar o operador. |

> 🐛 **Defeitos e códigos mortos identificados**
>
> - `ESCAPE BOTTOM` é usado em 7 blocos de validação (`L102, L108, L116, L122, L128, L134, L146, L152`) **fora de qualquer laço**. Em Natural padrão isso é erro de compilação. <!-- mystery: uso de ESCAPE BOTTOM fora de laço em CADBENEF; desconhecido se o dialeto local permite ou se o código nunca compilou como está -->
> - Por causa disso, o bloco `IF #ERRO / WRITE #MSG` em `L171-174` é **inalcançável** — `#MSG` nunca chega a ser exibido.
> - A idade é calculada por diferença de ano (`L155-159`), replicando o defeito da regra 18.
> - A VIEW declara `ENDERECO (A80)`, mas o DDM tem um **grupo** `BA GRP-ENDERECO` com 10 subcampos. Ver M-12.

## Regras de `CADDEPEND.NSN`

**Programa:** `01-arqueologia/legado-sifap/natural-programs/CADDEPEND.NSN` (133 linhas) · Par 1 · Visão
**Cabeçalho:** 1998 criação (Ana Lúcia Pereira) · 2008 "AJUSTE PE GROUP"
**Acesso a dados:** lê e escreve o grupo periódico de dependentes em `BENEFICIARIO` (FNR 150)

| #   | Declaração da Regra | Candidata EARS | Fonte | Classificação | Observações |
| --- | --- | --- | --- | --- | --- |
| 33 | Se o CPF do titular não existir, então o sistema deve recusar a inclusão de dependentes. | Unwanted | `CADDEPEND.NSN:L45-54` | Inferida | — |
| 34 | Se o titular estiver com situação `'C'` ou `'D'`, então o sistema deve recusar a inclusão de dependentes. | Unwanted | `CADDEPEND.NSN:L56-59` | Inferida | ⚠️ **Não bloqueia situação `'S'` (suspenso) nem `'I'` (inativo)**. Combinado com a regra 30, é possível incluir dependentes num beneficiário suspenso por idade. |
| 35 | Enquanto a quantidade de dependentes for igual ou inferior a 5, o sistema deve permitir a inclusão de novos dependentes. | State-driven | `CADDEPEND.NSN:L62-66` | **Mistério** | 🚨 Ver M-06 e M-15. O teste é `> 5`, então **o limite efetivo é 6**, não 5 — erro de contagem clássico. |
| 36 | Se o nome do dependente não for informado, então o sistema deve recusar. | Unwanted | `CADDEPEND.NSN:L79-82` | Inferida | — |
| 37 | Se o grau de parentesco não for `'FI'`, `'CO'`, `'IR'` ou `'OU'`, então o sistema deve recusar. | Unwanted | `CADDEPEND.NSN:L84-88` | **Mistério** | 🚨 O DDM define um conjunto **totalmente diferente**: `FI=FILHO CJ=CONJ NT=NETO TU=TUTEL`. Ver M-16. |
| 38 | Se o CPF do dependente já constar entre os dependentes do titular, então o sistema deve recusar a duplicidade. | Unwanted | `CADDEPEND.NSN:L94-103` | Inferida | ⚠️ A condição exige `CPF-DEP NE 0`. Dependentes **sem CPF não são verificados** — podem ser cadastrados infinitas vezes (até o limite de 6). |
| 39 | Quando um dependente válido for informado, o sistema deve gravá-lo na próxima posição livre do grupo e incrementar o contador. | Event-driven | `CADDEPEND.NSN:L109-123` | Inferida | Não grava `SIT-DEPENDENTE` nem `IND-DEFICIENCIA`, que existem no DDM. Sem auditoria (E-03). |
| 40 | Quando o operador responder diferente de `'S'`, o sistema deve encerrar o laço de inclusão. | Event-driven | `CADDEPEND.NSN:L125-128` | Inferida | — |

> 🐛 **Não há validação de idade, de situação nem de duplicidade contra outros titulares.**
> Um mesmo CPF de dependente pode ser vinculado a vários titulares diferentes — nada impede.
> <!-- mystery: CADDEPEND não verifica se o CPF do dependente já está vinculado a outro titular nem se ele é titular de benefício próprio; desconhecido se há controle em outro ponto do sistema -->

## Regras de `CADPROG.NSN`

**Programa:** `01-arqueologia/legado-sifap/natural-programs/CADPROG.NSN` (~125 linhas) · Par 1 · Visão
**Cabeçalho:** 1997 criação (Marcos Antônio Ribeiro) · **2003 "INC FATOR CORRECAO"** · 2012 novos códigos de elegibilidade
**Acesso a dados:** lê e escreve `PROGRAMA-SOCIAL` (FNR 151)

| #   | Declaração da Regra | Candidata EARS | Fonte | Classificação | Observações |
| --- | --- | --- | --- | --- | --- |
| 41 | Se a operação não for `'I'` (inclusão) nem `'C'` (consulta), então o sistema deve recusar. | Unwanted | `CADPROG.NSN:L51-54` | Inferida | Não há alteração nem exclusão de programa — **tabela paramétrica só cresce**. |
| 42 | Se o código do programa já existir, então o sistema deve recusar a inclusão. | Unwanted | `CADPROG.NSN:L81-84` | Inferida | — |
| 43 | **Quando um programa for incluído, o sistema deve gravar como valor-base o resultado de `valor informado × (1 + fator de reajuste × 0,347215)`.** | Event-driven | `CADPROG.NSN:L86-88` | **Mistério** | 🚨🚨🚨 **O "FATOR-K" foi localizado.** Ver M-17. |
| 44 | Quando um programa for incluído, o sistema deve atribuir situação `'A'` (ativa). | Event-driven | `CADPROG.NSN:L98` | Inferida | Fixo em código. Não há como cadastrar programa já encerrado. |
| 45 | Quando a operação for consulta, o sistema deve exibir os dados do programa; se não houver registro, deve informar "PROGRAMA NAO ENCONTRADO". | Event-driven | `CADPROG.NSN:L56-58, L110-124` | Inferida | Usa `*NUMBER(PROGRAMA-V) = 0` — idioma Natural correto, ao contrário do padrão de flag manual usado nos outros programas. |

> 🐛 **`CADPROG` não valida o tipo do programa.** O campo `#TIPO` é lido do operador e gravado
> sem nenhuma verificação contra `A`/`P`/`T` (`CADPROG.NSN:L65, L92`). Um erro de digitação aqui
> faz `VALELEG` cair no branch `NONE` (regra 15) e **reprovar 100% dos candidatos** ao programa,
> sem que ninguém entenda o motivo.

> 🐛 **O valor-base original é perdido.** O programa grava `#VLR-CALC` (já multiplicado) no campo
> `VLR-BASE`. O valor que o gestor digitou **não é armazenado em lugar nenhum**. Não há como
> auditar nem reverter a aplicação do fator.

## Regras de `VALBENEF.NSN`

**Programa:** `01-arqueologia/legado-sifap/natural-programs/VALBENEF.NSN` (~300 linhas) · Par 4 · Qualidade
**Cabeçalho:** 1998 criação (Márcia Helena Oliveira) · 2005 ajuste CPF · 2010 inclusão de validação de nome
**Acesso a dados:** declara a VIEW de `BENEFICIARIO`, mas **nunca executa `FIND`, `STORE` ou `UPDATE`**

| #   | Declaração da Regra | Candidata EARS | Fonte | Classificação | Observações |
| --- | --- | --- | --- | --- | --- |
| 46 | Se o CPF tiver todos os dígitos iguais, então o sistema deve considerá-lo inválido. | Unwanted | `VALBENEF.NSN:L186-195` | Inferida | Verificação que **falta** em `CADBENEF` e `VALDOCS` (E-02). |
| 47 | **Onde o CPF tiver todos os dígitos iguais e os três primeiros forem zero, o sistema deve considerá-lo válido.** | Optional | `VALBENEF.NSN:L195-200` | **Mistério** | 🚨 Ver M-18. Como a condição exige "todos iguais", o único valor que a satisfaz é `00000000000`. Comentário no código: "TESTE GOVERNO". |
| 48 | Se o dígito verificador do CPF não conferir, então o sistema deve registrar erro "CPF INVALIDO". | Unwanted | `VALBENEF.NSN:L114-120` | **Confirmada** | RN-001. |
| 49 | Se o ano de nascimento for anterior a 1900 ou posterior ao ano corrente, então o sistema deve registrar data inválida. | Unwanted | `VALBENEF.NSN:L253-256` | Inferida | — |
| 50 | Se o mês não estiver entre 1 e 12, ou o dia exceder o número de dias do mês, então o sistema deve registrar data inválida. | Unwanted | `VALBENEF.NSN:L257-263` | **Mistério** | 🚨 A tabela fixa fevereiro em **29 dias** (`L106`, comentário "CONSIDERA BISSEXTO"). O sistema aceita **29/02 em qualquer ano**. |
| 51 | Se o nome não contiver ao menos um espaço, então o sistema deve registrar "NOME INVALIDO - DEVE TER NOME E SOBRENOME". | Unwanted | `VALBENEF.NSN:L134-140, L268-283` | Inferida | Regra ausente em `CADBENEF` (regra 24) — validação inconsistente entre programas. |
| 52 | Onde a UF for informada, se não constar na tabela das 27 unidades federativas, então o sistema deve registrar "UF INVALIDA". | Optional | `VALBENEF.NSN:L60, L145-160` | Inferida | Tabela fixa em código (`L64-90`). UF em branco **passa sem validação**. |
| 53 | Se a situação não for `'A'`, `'S'`, `'C'`, `'I'` ou `'D'`, então o sistema deve registrar "STATUS INVALIDO". | Unwanted | `VALBENEF.NSN:L165-171` | **Confirmada** | Bate exatamente com o DDM (`CE SIT-BENEFICIARIO`). **Confirma que o valor `'E'` da RN-011 não existe.** |
| 54 | O sistema deve avaliar todas as validações e devolver `'V'` (válido) ou `'I'` (inválido) com a lista de erros. | Ubiquitous | `VALBENEF.NSN:L110-112, L175-179` | Inferida | — |

> 🐛 **`VALBENEF` não tem `INPUT` nem `PARAMETER`.** Todas as variáveis são `LOCAL` e nenhuma é
> preenchida. Executado como está, o programa **sempre valida zeros e brancos** — e sempre reprova.
> Como também não existe `CALLNAT` no codebase (E-01), não há como alimentá-lo.
> <!-- mystery: VALBENEF declara tudo como LOCAL, não possui INPUT nem PARAMETER e não é chamado por ninguém; desconhecido como o programa recebia dados em produção -->

## Regras de `VALDOCS.NSN`

**Programa:** `01-arqueologia/legado-sifap/natural-programs/VALDOCS.NSN` (~185 linhas) · Par 4 · Qualidade
**Cabeçalho:** 1998 criação (Ana Lúcia Pereira) · 2003 validação de RG · **2011 "AJUSTE CHECK ESPEC" (Roberto Mendes)**
**Acesso a dados:** declara a VIEW de `BENEFICIARIO` incluindo `DOCUMENTOS-OK`, mas **nunca lê nem grava**

| #   | Declaração da Regra | Candidata EARS | Fonte | Classificação | Observações |
| --- | --- | --- | --- | --- | --- |
| 55 | Se o CPF for zero ou tiver dígito verificador inválido, então o sistema deve registrar "CPF INVALIDO". | Unwanted | `VALDOCS.NSN:L68-73, L98-146` | **Confirmada** | RN-001. Terceira cópia do algoritmo (E-02) — **sem** a checagem de dígitos repetidos. |
| 56 | Se o RG estiver em branco ou tiver menos de 5 caracteres, então o sistema deve registrar "RG INVALIDO OU FORMATO INCORRETO". | Unwanted | `VALDOCS.NSN:L78-83, L150-166` | Inferida | O comprimento é medido pela posição do primeiro espaço; se não houver espaço, **assume 15**. RG sem espaços nunca é reprovado por tamanho. |
| 57 | **Onde os três primeiros dígitos do CPF constarem na tabela de prefixos especiais, o sistema deve marcar o documento como válido e descartar todos os erros já registrados.** | Optional | `VALDOCS.NSN:L49-56, L88, L168-181` | **Mistério** | 🚨🚨 Ver M-19. Executa **depois** das validações e as anula. |
| 58 | Quando a validação terminar, o sistema deve exibir o resultado e a lista de erros remanescentes. | Event-driven | `VALDOCS.NSN:L91-99` | Inferida | Saída só em terminal. Nada é persistido. |

> 🐛 **`DOCUMENTOS-OK` nunca é gravado.** O campo está na VIEW (`VALDOCS.NSN:L17`) e nunca recebe
> valor. É exatamente o campo que `VALELEG` lê na regra 12 para reprovar por "DOCUMENTACAO
> INCOMPLETA" — e que também **não existe no DDM**. Fecha o ciclo de M-04.

### Mistérios abertos do Lote A

| ID | Pergunta em aberto | Evidência | Impacto |
| --- | --- | --- | --- |
| M-13 | Por que `CADBENEF` rejeita sexo `'I'` se o DDM o prevê como "indefinido"? | `CADBENEF.NSN:L131-135`; `BENEFICIARIO.ddm` campo `AG SEXO` | 🟡 Médio. Registros com `'I'` existentes na base não podem mais ser alterados. |
| M-14 | Qual a base legal da suspensão automática acima de 75 anos, e ela é intencional em alterações? | `CADBENEF.NSN:L166-169`; cabeçalho "2011 - AJUSTE STATUS IDOSO" | 🔴 **Crítico.** Encadeado com a regra 5, remove o benefício de todo idoso acima de 75 anos. Nenhuma documentação menciona isso. |
| M-15 | O limite de dependentes é 3 (RN-004), 5 (intenção do código), 6 (comportamento real) ou 10 (grupo `PE`)? | `CADDEPEND.NSN:L62-66`; `BENEFICIARIO.ddm` `1 DA GRP-DEPENDENTE PE ... 10`; RN-004 | 🟠 Alto. Substitui e detalha M-06. |
| M-16 | Qual conjunto de códigos de parentesco é o correto: `FI/CO/IR/OU` (código) ou `FI/CJ/NT/TU` (DDM)? | `CADDEPEND.NSN:L84-88`; `BENEFICIARIO.ddm` campo `DE PARENTESCO` | 🟠 Alto. Dados gravados podem ser ilegíveis para quem lê pelo DDM. |
| M-17 | O que é a constante `0,347215` e por que ela multiplica o fator de reajuste? | `CADPROG.NSN:L86-88`; `PROGRAMA-SOCIAL.ddm` campo `BG FATOR-K` (`>>> NAO DOCUMENTADO <<<`, inserido ago/2008 por Adilson, "ATENDE SOLICITACAO SENARC"); `REGRAS-NEGOCIO-2012.md` §2.1 nota | 🔴 **Crítico.** É o "FATOR-K" que a documentação de 2012 declarou não conseguir explicar. Afeta o valor-base de todos os 45 programas. Nota adicional: o código calcula com precisão `N5.6`, o DDM armazena `N5.4` — e **o valor calculado nunca é gravado no campo `FATOR-K`**. |
| M-18 | Por que o CPF `000.000.000-00` é aceito como válido? | `VALBENEF.NSN:L195-200` | 🔴 **Crítico.** Backdoor de teste em código de produção. |
| M-19 | Quem autorizou os 8 prefixos de CPF que anulam toda a validação documental, e por que a lista inclui `100` e `999`? | `VALDOCS.NSN:L49-56, L168-181`; cabeçalho "2011 - AJUSTE CHECK ESPEC" | 🔴 **Crítico.** Prefixos como `100` cobrem uma faixa ampla de CPFs reais. Anula inclusive erros já detectados. |

---

## Regras de `BATCHPGT.NSN`

**Programa:** `01-arqueologia/legado-sifap/natural-programs/BATCHPGT.NSN` (~360 linhas) · Par 2 · Arquitetura
**Cabeçalho:** 1997 criação · 2000 "OTIMIZ ORD CPF" · 2004 log de erros · 2009 "AJUSTE 13O/ABONO" · 2012 novas faixas · **2015 "INC AUDITORIA"**
**Acesso a dados:** lê `BENEFICIARIO` (150) e `PROGRAMA-SOCIAL` (151); lê e **escreve** `PAGAMENTO` (152)

> 🚨 **O cabeçalho mente.** A linha 14 declara "CHAMA CALCBENF E CALCDSCT" e a documentação §5.1
> repete a afirmação. **Não existe nenhuma chamada.** O batch reimplementa a lógica de cálculo
> internamente — o próprio comentário da linha 123 admite: `(MESMA DO CALCBENF)`.
> Existem, portanto, **duas implementações paralelas** da fórmula de benefício que podem divergir
> a qualquer manutenção. Ver M-20.

> 🚨 **O cabeçalho mente de novo.** A alteração de 2015 diz "INC AUDITORIA". O programa **não
> declara a VIEW de `AUDITORIA` nem grava um único registro** de trilha, apesar de fazer `STORE`
> em produção. Ver M-21.

| #   | Declaração da Regra | Candidata EARS | Fonte | Classificação | Observações |
| --- | --- | --- | --- | --- | --- |
| 59 | Quando o processamento iniciar, o sistema deve derivar a competência a partir do ano e mês da data corrente. | Event-driven | `BATCHPGT.NSN:L107-110` | Inferida | A competência **não é parametrizável**. Reprocessar mês anterior é impossível sem alterar a data do sistema. |
| 60 | O sistema deve processar os beneficiários em ordem crescente de CPF. | Ubiquitous | `BATCHPGT.NSN:L176-182` | **Mistério** | 🚨 A doc §5.1 afirma que a ordenação é **alfabética por nome** e alerta para acumuladores por faixa alfabética. O código lê `BY CPF` desde 2000. Comentário na linha 179: "SISTEMAS DOWNSTREAM DEPENDEM DESTA ORDENACAO". Ver M-22. |
| 61 | Se o CPF for igual ao do registro anterior, então o sistema deve ignorar o registro. | Unwanted | `BATCHPGT.NSN:L187-192` | Inferida | ⚠️ Um beneficiário com dois vínculos de programa recebe **um único pagamento** — o do primeiro registro lido. |
| 62 | Se a situação do beneficiário não for `'A'`, então o sistema deve ignorá-lo. | Unwanted | `BATCHPGT.NSN:L194-198` | **Confirmada** | Doc §5.1: "todos os beneficiários ativos são processados". Encadeia com a regra 30 — **quem passou de 75 anos é excluído do pagamento**. |
| 63 | Se já existir pagamento do beneficiário na competência corrente, então o sistema deve ignorá-lo. | Unwanted | `BATCHPGT.NSN:L200-210` | Inferida | Idempotência por competência. |
| 64 | Se o programa do beneficiário não existir, então o sistema deve registrar erro e prosseguir. | Unwanted | `BATCHPGT.NSN:L212-226` | **Confirmada** | Doc §5.2: "erros individuais não interrompem o processamento". |
| 65 | Se o programa não estiver ativo, então o sistema deve ignorar o beneficiário. | Unwanted | `BATCHPGT.NSN:L227-230` | Inferida | Contabilizado como "ignorado", não como erro — some do relatório de erros. |
| 66 | Onde o código de região estiver entre 1 e 25, o sistema deve aplicar o fator regional da tabela; caso contrário deve aplicar fator 1,0. | Optional | `BATCHPGT.NSN:L239-244` | **Mistério** | 🚨 A tabela tem **27 posições** (`L124-150`) mas só 25 são alcançáveis — as posições 26 e 27 são **código morto**. E o DDM define região como `01-05 ou 99`. Ver M-23. |
| 67 | O sistema deve aplicar fator familiar de 1,0 sem dependentes; 1,0 + 0,05 por dependente até 2; 1,10 + 0,03 por dependente adicional até 4; e 1,16 + 0,02 acima disso. | Ubiquitous | `BATCHPGT.NSN:L246-259` | Inferida | Escada de 3 faixas, sem suporte documental. RN-013 descreve um **acréscimo fixo por dependente**, não um multiplicador. |
| 68 | O sistema deve aplicar fator de renda conforme a primeira faixa cujo teto seja maior ou igual à renda familiar (300 → 1,0; 600 → 0,85; 1.000 → 0,70; 1.500 → 0,55; 9.999,99 → 0,40). | Ubiquitous | `BATCHPGT.NSN:L152-162, L261-262` | **Confirmada (parcial)** | RN-018 descreve exatamente esse mecanismo de "primeira faixa cujo limite superior seja ≥ renda" — mas fala em **renda per capita**, e o código usa renda **familiar total** (mesmo defeito de M-02). ⚠️ Renda acima de 9.999,99 deixa `#FATOR-RND` **sem valor atribuído**. |
| 69 | O sistema deve aplicar fator etário de 1,15 a partir de 65 anos; 1,10 a partir de 60; 1,05 abaixo de 18; e 1,0 nos demais casos. | Ubiquitous | `BATCHPGT.NSN:L264-277` | Inferida | Sem suporte documental. Note que menores de 18 recebem majoração — mas RN-006 veda o cadastro de menores de 16. |
| 70 | O sistema deve calcular o benefício como valor-base multiplicado pelos fatores regional, familiar, de renda e etário, e em seguida pelo fator de reajuste. | Ubiquitous | `BATCHPGT.NSN:L279-282` | **Mistério** | 🚨 **Reajuste aplicado duas vezes.** `CADPROG` já gravou o valor-base multiplicado por `(1 + reajuste × 0,347215)` (regra 43); aqui ele é multiplicado novamente por `(1 + reajuste)`. Ver M-24. |
| 71 | O sistema deve truncar todos os valores monetários em centavos, sem arredondamento. | Ubiquitous | `BATCHPGT.NSN:L283-285, L295-296, L300-301, L310-311, L319-320` | **Confirmada** | RN-014: "sempre arredondado para baixo (truncamento)". Implementado por variável inteira intermediária `#VLR-TEMP (N11)`. |
| 72 | Onde a competência for dezembro, o sistema deve calcular um 13º valor usando apenas os fatores regional e etário, e somá-lo ao bruto. | Optional | `BATCHPGT.NSN:L291-297` | **Mistério** | 🚨 O 13º **ignora** o fator familiar, o fator de renda e o reajuste. RN-006 da lista de pendências marca este cálculo como "Alta prioridade — não documentado". Ver M-25. |
| 73 | Onde a competência for dezembro e o programa for do tipo assistencial, o sistema deve acrescentar abono de 15% sobre o benefício mensal. | Optional | `BATCHPGT.NSN:L298-303` | **Mistério** | Percentual fixo em código, sem documentação. Beneficia apenas o tipo `'A'`. |
| 74 | Se o valor bruto for superior a 500,00, então o sistema deve aplicar desconto de 3%. | Unwanted | `BATCHPGT.NSN:L306-312` | **Mistério** | 🚨 Comentário no código: "CALC DESCONTOS SIMPLIFICADO". Ignora completamente `CALCDSCT`, os tipos de desconto da RN-022 e o teto de 30% da RN-021. Ver M-20. |
| 75 | O sistema deve calcular o líquido como bruto menos descontos, nunca inferior a zero. | Ubiquitous | `BATCHPGT.NSN:L314-320` | Inferida | Piso em zero mascara erro de cálculo em vez de sinalizá-lo. |
| 76 | Quando o cálculo terminar, o sistema deve gravar o pagamento com situação `'G'` (gerado). | Event-driven | `BATCHPGT.NSN:L322-336` | **Mistério** | 🚨 Doc §5.1 afirma que o registro é gravado com status `'P'` (pendente). O código grava `'G'`. Ambos existem no DDM. Ver M-26. |
| 77 | O sistema deve numerar o pagamento a partir do maior número existente, incrementando de um. | Ubiquitous | `BATCHPGT.NSN:L170-174, L323` | **Mistério** | 🚨 Sem reserva nem bloqueio. Duas execuções simultâneas geram **números duplicados**. Ver M-27. |
| 78 | Ao final, o sistema deve exibir totais de processados, gerados, ignorados, erros e somatórios de valores. | Ubiquitous | `BATCHPGT.NSN:L351-365` | Inferida | ⚠️ Não há o limite `MAX-ERROS` (default 100) com `ABEND U4038` descrito na doc §5.2, nem a geração do arquivo de remessa CNAB 240 descrita na §5.1. |

> 🐛 **Ausências relevantes:** o programa não gera arquivo de remessa CNAB 240, não interrompe por
> excesso de erros, não grava auditoria e não lê os campos `IND-EXIGE-*` do DDM. As estruturas
> `#LOG-WORK` / `#LOG-ERRO` (`L100-102`) são declaradas e **nunca usadas**.

## Regras de `BATCHCON.NSN`

**Programa:** `01-arqueologia/legado-sifap/natural-programs/BATCHCON.NSN` (~290 linhas) · Par 2 · Arquitetura
**Cabeçalho:** 2000 criação (Marcos Antônio Ribeiro) · 2005 "INC BANCO REAL" · 2008 ajuste CNAB 240 · 2014 inclusão de auditoria
**Acesso a dados:** lê arquivo de trabalho CNAB 240; lê e atualiza `PAGAMENTO` (152); **escreve** `AUDITORIA` (153)

| #   | Declaração da Regra | Candidata EARS | Fonte | Classificação | Observações |
| --- | --- | --- | --- | --- | --- |
| 79 | Se o tipo do registro CNAB não for `'3'` (detalhe), então o sistema deve ignorá-lo. | Unwanted | `BATCHCON.NSN:L110-117` | Inferida | Header, trailer e lotes descartados sem conferência de totais. |
| 80 | O sistema deve extrair CPF, valor, data de pagamento, código de retorno e número do documento de posições fixas do registro CNAB 240. | Ubiquitous | `BATCHCON.NSN:L119-130` | Inferida | Posições fixas em código. Qualquer mudança de layout do banco quebra silenciosamente. |
| 81 | O sistema deve converter o valor recebido de centavos para reais dividindo por 100. | Ubiquitous | `BATCHCON.NSN:L132-135` | Inferida | Conversão alfanumérico → numérico sem tratamento de erro. |
| 82 | Se não houver pagamento com o mesmo número, CPF e competência, então o sistema deve registrar "NAO ENCONTRADO" e prosseguir. | Unwanted | `BATCHCON.NSN:L140-156` | Inferida | ⚠️ Divergência de chave: o retorno é casado por número de documento, mas o registro não recebe marcação — some do controle. |
| 83 | Se a diferença absoluta entre o valor do SIFAP e o valor do banco exceder R$ 0,01, então o sistema deve registrar divergência e gravar auditoria. | Unwanted | `BATCHCON.NSN:L158-170` | **Mistério** | 🚨 Em caso de divergência o pagamento **não tem a situação alterada** — permanece como estava, sem sinalização no próprio registro. Só existe rastro na auditoria. Ver M-28. |
| 84 | Onde o código de retorno for `'00'`, o sistema deve marcar o pagamento com situação `'P'`. | Optional | `BATCHCON.NSN:L174-182` | **Mistério** | 🚨 No DDM, `P = PENDENTE`. Retorno `'00'` é sucesso bancário. O código marca o pagamento **bem-sucedido como pendente**. Ver M-29. |
| 85 | Onde o código de retorno for `'01'`, o sistema deve marcar o pagamento como `'D'` (devolvido). | Optional | `BATCHCON.NSN:L183-189` | Inferida | Consistente com o DDM. |
| 86 | Onde o código de retorno for `'02'`, o sistema deve marcar o pagamento como `'E'`. | Optional | `BATCHCON.NSN:L190-196` | **Mistério** | No DDM, `E = EMITIDO`. Retorno `'02'` aparenta ser erro. Mesma confusão semântica de M-29. |
| 87 | Se o código de retorno não for `'00'`, `'01'` nem `'02'`, então o sistema deve apenas exibir aviso e não alterar o pagamento. | Unwanted | `BATCHCON.NSN:L197-200` | Inferida | Retorno desconhecido é **contabilizado como conciliado** — o `ADD 1 TO #QTD-CONCILIADOS` ocorre antes do `DECIDE`. |
| 88 | Quando um pagamento for conciliado, o sistema deve gravar registro de auditoria com ação `'CO'`. | Event-driven | `BATCHCON.NSN:L201, L245-259` | **Mistério** | 🚨 No DDM, `CO = CONSULTA`, e a nota registra que ações `'CO'` **não são gravadas desde 2010** por volume. O programa usa `'CO'` para "conciliado" — colisão de código. Ver M-30. |
| 89 | Quando uma divergência for detectada, o sistema deve gravar auditoria com ação `'DV'`, valor do SIFAP e valor do banco. | Event-driven | `BATCHCON.NSN:L167, L261-278` | **Mistério** | O código `'DV'` **não consta na lista de ações do DDM** (`IN/AL/EX/CO/LG/LO/BT/ER/AU/RE`). |
| 90 | Ao final, o sistema deve exibir totais de lidos, conciliados, divergentes, não encontrados e registros de auditoria. | Ubiquitous | `BATCHCON.NSN:L230-240` | Inferida | — |

> 🪦 **Código morto preservado deliberadamente.** As linhas `L215-228` contêm toda a integração com
> o Banco Real, comentada, com a justificativa "BANCO REAL FOI ADQUIRIDO PELO SANTANDER EM 2007 —
> MANTER CODIGO PARA REFERENCIA HISTORICA". A sub-rotina `CONCILIA-REAL` referenciada **não existe**.
> A doc §6 registra que as regras de conciliação nunca foram levantadas porque a responsável foi
> transferida.

## Regras de `CONSBENF.NSN`

**Programa:** `01-arqueologia/legado-sifap/natural-programs/CONSBENF.NSN` (~195 linhas) · Par 5 · Operações
**Cabeçalho:** 1998 criação (Márcia Helena Oliveira) · **2003 "INC MASCARA CPF"** · 2007 histórico de pagamentos · 2012 ajuste de tela
**Acesso a dados:** lê `BENEFICIARIO` (150) e `PAGAMENTO` (152). **Somente leitura.**

| #   | Declaração da Regra | Candidata EARS | Fonte | Classificação | Observações |
| --- | --- | --- | --- | --- | --- |
| 91 | Se a tela formatada não puder ser carregada, então o sistema deve apresentar entrada alternativa em modo texto. | Unwanted | `CONSBENF.NSN:L69-78` | **Mistério** | 🚨 O programa referencia o mapa `'CONSBENF-M01'`, e **não existe nenhum arquivo `.map` no repositório** — confirmando a ausência sinalizada no inventário. Ver M-31. |
| 92 | Onde o tipo de busca não for informado, o sistema deve assumir busca por CPF. | Optional | `CONSBENF.NSN:L80-82` | Inferida | — |
| 93 | Onde o tipo de busca for `'C'`, o sistema deve localizar o beneficiário por CPF; onde for `'N'`, por NIS. | Optional | `CONSBENF.NSN:L86-94` | Inferida | Único programa que consulta por NIS. Reforça M-05 — `NIS` não existe no DDM. |
| 94 | Se o tipo de busca não for `'C'` nem `'N'`, então o sistema deve recusar a consulta. | Unwanted | `CONSBENF.NSN:L95-97` | Inferida | — |
| 95 | Se o beneficiário não for encontrado, então o sistema deve informar e encerrar. | Unwanted | `CONSBENF.NSN:L100-103` | Inferida | — |
| 96 | Quando os dados forem exibidos, o sistema deve mascarar o CPF no formato `***.***.NNN-NN`. | Event-driven | `CONSBENF.NSN:L104-107, L175-190` | **Mistério** | 🚨 Ver M-32 — a máscara vaza dados em um dos caminhos. |
| 97 | O sistema deve traduzir o código de situação para descrição legível, usando "DESCONHECIDO" para valores fora do domínio. | Ubiquitous | `CONSBENF.NSN:L109-123` | **Confirmada** | Domínio `A/S/C/I/D` idêntico ao DDM e a `VALBENEF` (regra 53). Terceira confirmação de que `'E'` da RN-011 não existe. |
| 98 | O sistema deve exibir os dados cadastrais e os 12 pagamentos mais recentes do beneficiário. | Ubiquitous | `CONSBENF.NSN:L142-163` | **Mistério** | 🚨 O rótulo diz "ÚLTIMOS 12", mas a leitura é feita na ordem do descritor de CPF, **sem ordenação por competência**. Os 12 exibidos são arbitrários, não os mais recentes. Ver M-33. |
| 99 | Se não houver pagamentos, o sistema deve informar "NENHUM PAGAMENTO ENCONTRADO". | Unwanted | `CONSBENF.NSN:L165-167` | Inferida | — |

> 🐛 **A máscara protege só o CPF.** Nome completo, endereço, CEP, renda familiar, número de
> dependentes e NIS são exibidos **em claro** (`L126-140`). Mascarar o CPF e revelar o NIS e a
> renda não reduz a exposição de dado pessoal sensível.

> 🐛 **Nenhuma consulta é auditada.** Coerente com a nota do DDM (`'CO'` não gravado desde 2010),
> mas significa que **não há rastro de quem consultou quais beneficiários**.

### Mistérios abertos do Lote B

| ID | Pergunta em aberto | Evidência | Impacto |
| --- | --- | --- | --- |
| M-20 | Existem duas implementações do cálculo de benefício e duas de desconto. Qual é a que vale? | `BATCHPGT.NSN:L14, L123, L279-312` vs `CALCBENF.NSN` e `CALCDSCT.NSN` | 🔴 **Crítico.** O batch que paga 4,2 milhões de pessoas usa a sua própria cópia. |
| M-21 | O que significa a entrada de changelog "2015 — INC AUDITORIA" em um programa que não grava auditoria? | `BATCHPGT.NSN:L10`; ausência de VIEW de `AUDITORIA` | 🟠 Alto. Changelog fantasma; sugere alteração revertida sem atualizar o cabeçalho. |
| M-22 | A ordenação do batch é por CPF (código) ou alfabética por nome (doc §5.1)? Quais são os "sistemas downstream" que dependem dela? | `BATCHPGT.NSN:L176-182`; `REGRAS-NEGOCIO-2012.md` §5.1 e nota | 🟠 Alto. A doc alerta para acumuladores por faixa alfabética que não existem no código. |
| M-23 | Por que a tabela de fatores regionais tem 27 posições, o laço lê 25 e o DDM define apenas 01-05 e 99? | `BATCHPGT.NSN:L124-150, L239-244`; `BENEFICIARIO.ddm` campo `BJ`; `PROGRAMA-SOCIAL.ddm` grupo `FA` | 🔴 **Crítico.** Beneficiários da região 99 recebem fator 1,0 e **perdem** o complemento regional que o DDM prevê para a região especial. |
| M-24 | O fator de reajuste deve ser aplicado no cadastro do programa, no cálculo do pagamento, ou nos dois? | `CADPROG.NSN:L86-88` + `BATCHPGT.NSN:L282`; RN-019 e RN-020 | 🔴 **Crítico.** Aplicação dupla com fórmulas diferentes infla o benefício de forma não auditável. |
| M-25 | Por que o 13º ignora fator familiar, fator de renda e reajuste? | `BATCHPGT.NSN:L291-297`; `REGRAS-NEGOCIO-2012.md` §6 (marcado "Alta prioridade") | 🟠 Alto. |
| M-26 | A situação inicial do pagamento é `'G'` (código) ou `'P'` (doc §5.1)? | `BATCHPGT.NSN:L331`; doc §5.1 | 🟠 Alto. Define o que a conciliação deve procurar. |
| M-27 | Como o sistema evita numeração duplicada de pagamento em execuções concorrentes? | `BATCHPGT.NSN:L170-174, L323`; mesma técnica em `BATCHCON.NSN:L94-98` | 🔴 **Crítico.** 180 milhões de registros sem chave garantidamente única. |
| M-28 | Por que um pagamento divergente não recebe marcação no próprio registro? | `BATCHCON.NSN:L158-170` | 🟠 Alto. Divergência só existe na auditoria; o pagamento parece normal. |
| M-29 | O que significam `'P'` e `'E'` na conciliação, se no DDM são "pendente" e "emitido"? | `BATCHCON.NSN:L174-196`; `PAGAMENTO.ddm` campo `DA` | 🔴 **Crítico.** Pagamento confirmado pelo banco fica marcado como pendente. |
| M-30 | O código de ação `'CO'` significa "consulta" (DDM) ou "conciliado" (BATCHCON)? E o que é `'DV'`? | `BATCHCON.NSN:L251, L267`; `AUDITORIA.ddm` campo `BA` e nota de 2010 | 🟠 Alto. Contamina a trilha de auditoria legal (IN-TCU 63/2010). |
| M-31 | Onde estão os arquivos `.map` referenciados pelo código? | `CONSBENF.NSN:L69`; `inventory.md` (nenhum `.map` no repositório) | 🟡 Médio. Confirma a ausência sinalizada no inventário. |
| M-32 | Por que a máscara de CPF revela os três primeiros dígitos quando o CPF tem menos de 11 posições? | `CONSBENF.NSN:L175-190` e comentário `L169-174` ("NAO CORRIGIR SEM APROVACAO DA AUDITORIA") | 🔴 **Crítico.** Vazamento de dado pessoal em caminho conhecido e deliberadamente não corrigido. |
| M-33 | Os "últimos 12 pagamentos" são realmente os mais recentes? | `CONSBENF.NSN:L142-163` | 🟠 Alto. Leitura sem ordenação por competência. |

---

## Regras de `CALCBENF.NSN`

**Programa:** `01-arqueologia/legado-sifap/natural-programs/CALCBENF.NSN` (~300 linhas) · Par 3 · Implementação
**Cabeçalho:** 1997 criação · 2001 13º salário · 2004 ajuste fator regional · 2009 abono natalino · 2013 novas faixas de renda
**Acesso a dados:** lê `BENEFICIARIO` (150) e `PROGRAMA-SOCIAL` (151); **escreve** `PAGAMENTO` (152)

| #   | Declaração da Regra | Candidata EARS | Fonte | Classificação | Observações |
| --- | --- | --- | --- | --- | --- |
| 100 | Se o mês da competência não estiver entre 1 e 12, então o sistema deve recusar o cálculo. | Unwanted | `CALCBENF.NSN:L146-149` | Inferida | Não valida o ano. Competência `999901` é aceita. |
| 101 | Se o beneficiário não estiver com situação `'A'`, então o sistema deve recusar o cálculo. | Unwanted | `CALCBENF.NSN:L163-166` | **Confirmada** | Coerente com a regra 62 do batch. |
| 102 | Onde o código de região estiver entre 1 e 25, o sistema deve aplicar o fator regional correspondente; caso contrário deve aplicar 1,0. | Optional | `CALCBENF.NSN:L88-119, L181-185` | **Mistério** | 🚨 Ver M-34. Aqui os comentários revelam o mapeamento UF por posição — e ele está **errado e incompleto**. |
| 103 | O sistema deve aplicar fator familiar em escada conforme a quantidade de dependentes. | Ubiquitous | `CALCBENF.NSN:L187-199` | Inferida | Idêntico à regra 67. Duplicação literal de `BATCHPGT`. |
| 104 | O sistema deve aplicar o fator da primeira faixa de renda cujo teto seja maior ou igual à renda familiar. | Ubiquitous | `CALCBENF.NSN:L122-132, L201-202` | **Confirmada (parcial)** | Mesmo defeito de renda total × per capita (M-02). Renda acima de 9.999,99 deixa o fator **sem valor**. |
| 105 | O sistema deve aplicar fator etário conforme as faixas 65+, 60+, menor de 18 e demais. | Ubiquitous | `CALCBENF.NSN:L204-217` | Inferida | Idêntico à regra 69. |
| 106 | O sistema deve calcular o benefício como base × fator regional × fator familiar × fator de renda × fator etário, e então aplicar o reajuste do programa. | Ubiquitous | `CALCBENF.NSN:L219-227` | **Mistério** | Mesma dupla aplicação de reajuste de M-24. |
| 107 | O sistema deve truncar valores monetários em centavos. | Ubiquitous | `CALCBENF.NSN:L229-231` | **Confirmada** | RN-014. |
| 108 | Onde a competência for dezembro, o sistema deve somar um 13º calculado como base × fator regional × fator etário. | Optional | `CALCBENF.NSN:L233-246` | **Mistério** | 🚨 O comentário das linhas 234-236 declara a fórmula como `VLR_BASE * FATOR_REG * (MESES_ATIVOS/12)` — **proporcional**. O código não calcula `MESES_ATIVOS` em lugar nenhum. Ver M-35. |
| 109 | Onde for dezembro e o programa for do tipo `'A'`, o sistema deve acrescentar abono de 15%. | Optional | `CALCBENF.NSN:L247-256` | **Mistério** | Igual à regra 73. |
| 110 | O sistema deve aplicar desconto de 3% quando o bruto exceder 500,00. | Unwanted | `CALCBENF.NSN:L258-259, L290-299` | **Mistério** | 🚨 O próprio comentário admite: "CALC DESCONTOS - SIMPLIFICADO (VER CALCDSCT P/ COMPLETO)". `CALCDSCT` **não é chamado** (E-01). Terceira cópia divergente da regra de desconto. |
| 111 | Quando o cálculo terminar, o sistema deve gravar um novo registro de pagamento com situação `'G'`. | Event-driven | `CALCBENF.NSN:L271-283` | **Mistério** | 🚨🚨 O `STORE` ocorre **sem atribuir `NUM-PAGTO`** e **sem verificar se já existe pagamento na competência**. Ver M-36. |

> 🐛 **Mapeamento regional incorreto** (`CALCBENF.NSN:L88-119`). O comentário da linha 87 declara
> `01-05=NORTE 06-10=NORDESTE 11-15=SUDESTE 16-20=SUL 21-25=C.OESTE`, mas os comentários por
> posição contradizem isso:
>
> - Posições 19 e 20 são `MS` e `MT` — **Centro-Oeste**, classificadas como Sul.
> - Posições 24 e 25 são `RR` (Norte) e `SE` (Nordeste), classificadas como Centro-Oeste.
> - A posição 15 é `REF` — **não é uma UF**, é um marcador de referência com fator 1,0.
> - **Faltam 3 estados:** `AL`, `PB` e `RN` não aparecem em nenhuma posição.
> - As posições 26 e 27 são marcadas `RESERVA` e são inalcançáveis (o laço vai até 25).

## Regras de `CALCDSCT.NSN`

**Programa:** `01-arqueologia/legado-sifap/natural-programs/CALCDSCT.NSN` (~215 linhas) · Par 3 · Implementação
**Cabeçalho:** 1999 criação (Roberto Mendes Junior) · **2007 "INC DESC JUDICIAL"** · 2015 novas alíquotas
**Acesso a dados:** lê `BENEFICIARIO` (150); lê e **atualiza** `PAGAMENTO` (152)

| #   | Declaração da Regra | Candidata EARS | Fonte | Classificação | Observações |
| --- | --- | --- | --- | --- | --- |
| 112 | Se o pagamento informado não existir ou não pertencer ao CPF informado, então o sistema deve recusar. | Unwanted | `CALCDSCT.NSN:L72-86` | Inferida | — |
| 113 | Se o beneficiário não existir, então o sistema deve recusar. | Unwanted | `CALCDSCT.NSN:L88-95` | Inferida | Usa `*NUMBER(...)`, idioma correto. |
| 114 | O sistema deve aplicar contribuição social obrigatória de 3%, 5%, 7% ou 9% conforme a faixa do valor bruto (500 / 1.000 / 2.000 / 9.999,99). | Ubiquitous | `CALCDSCT.NSN:L56-64, L99, L200-211` | Inferida | Sem suporte documental. RN-022 lista "contribuição previdenciária" sem alíquotas. Bruto acima de 9.999,99 **não recebe contribuição alguma**. |
| 115 | O sistema deve calcular o teto de desconto como 30% do valor bruto. | Ubiquitous | `CALCDSCT.NSN:L102-106` | **Confirmada** | RN-021: "o total de descontos não pode exceder 30% do valor bruto". |
| 116 | O sistema deve ignorar descontos cuja data de fim seja anterior à data corrente ou cuja data de início seja posterior a ela. | Ubiquitous | `CALCDSCT.NSN:L111-118` | **Mistério** | 🚨 A vigência é testada contra a **data de hoje**, não contra a competência do pagamento. Recalcular um pagamento antigo aplica os descontos vigentes hoje. Ver M-37. |
| 117 | Onde o desconto for judicial, o sistema deve usar o valor fixo quando informado, ou o percentual sobre o bruto. | Optional | `CALCDSCT.NSN:L122-133` | Inferida | — |
| 118 | **Onde o desconto for judicial, o sistema não deve aplicar o teto de 30%.** | Optional | `CALCDSCT.NSN:L130-131, L179-184` | **Confirmada** | 🎯 Comentário explícito: "JUDICIAL NAO TEM TETO". **Isto confirma a suspeita que a documentação de 2012 não conseguiu validar** (nota da RN-021: "Marcos Antônio mencionou que existe uma exceção para retenções judiciais... não foi possível confirmar no código"). |
| 119 | Onde o desconto for pensão alimentícia, imposto ou administrativo, o sistema deve calcular por valor fixo ou percentual e somar ao total. | Optional | `CALCDSCT.NSN:L134-160` | Inferida | — |
| 120 | Onde o desconto for sindical, o sistema deve aplicar 1% do valor bruto. | Optional | `CALCDSCT.NSN:L155-158` | **Mistério** | Percentual fixo em código; ignora o `PCT-DSCT` cadastrado. |
| 121 | Se o tipo de desconto for desconhecido, então o sistema deve ignorá-lo silenciosamente. | Unwanted | `CALCDSCT.NSN:L172-173` | Inferida | `NONE → IGNORE`. Desconto com tipo inválido some sem aviso. |
| 122 | Se o total de descontos não judiciais exceder o teto, então o sistema deve reduzir o total ao teto. | Unwanted | `CALCDSCT.NSN:L179-184` | **Mistério** | 🚨 O corte é aplicado ao **acumulado corrente dentro do laço**, então um desconto judicial já somado pode ser **apagado** por um desconto comum processado depois. RN-023 prevê descarte por prioridade, não truncamento do total. Ver M-38. |
| 123 | Quando o cálculo terminar, o sistema deve atualizar o valor de desconto do pagamento. | Event-driven | `CALCDSCT.NSN:L191-196` | **Mistério** | 🚨 **O valor líquido não é recalculado.** O pagamento fica com `VLR-LIQUIDO` inconsistente com `VLR-BRUTO − VLR-DESCONTO`. Ver M-39. |

> 🐛 **O grupo de descontos está no arquivo errado.** A VIEW declara `DESCONTOS (PE)` dentro de
> `BENEFICIARIO` (`L24-31`). No DDM, o grupo periódico `CA GRP-DESCONTO` pertence a **`PAGAMENTO`**,
> e `BENEFICIARIO.ddm` **não tem nenhum grupo de descontos**.

> 🐛 **Quatro taxonomias incompatíveis para tipo de desconto:**
>
> | Fonte | Formato | Valores |
> | --- | --- | --- |
> | `CALCDSCT.NSN:L26-27` | `A1` | `C` `I` `J` `S` `P` `A` |
> | `PAGAMENTO.ddm` campo `CB` | `A3` | `IR` `JD` `CS` `PA` `EM` `TX` `OU` `EX` |
> | `PROGRAMA-SOCIAL.ddm` campo `EA` | `A3` (MU) | idem acima |
> | `REGRAS-NEGOCIO-2012.md` RN-022 | numérico | `01` a `05` (com `05` marcado "A COMPLETAR") |

## Regras de `CALCCORR.NSN`

**Programa:** `01-arqueologia/legado-sifap/natural-programs/CALCCORR.NSN` (~215 linhas) · Par 3 · Implementação
**Cabeçalho:** 2001 criação (Patrícia Gomes de Souza) · 2006 novos índices IPCA · 2014 ajuste de período
**Acesso a dados:** lê e **atualiza** `PAGAMENTO` (152)

| #   | Declaração da Regra | Candidata EARS | Fonte | Classificação | Observações |
| --- | --- | --- | --- | --- | --- |
| 124 | Se a competência inicial for maior que a final, então o sistema deve recusar o processamento. | Unwanted | `CALCCORR.NSN:L120-123` | Inferida | — |
| 125 | O sistema deve processar apenas os pagamentos do CPF informado dentro do intervalo de competências. | Ubiquitous | `CALCCORR.NSN:L129-141` | **Mistério** | 🚨 A leitura é ordenada por CPF, não por competência, mas o laço usa `ESCAPE BOTTOM` ao encontrar competência maior que o fim. Se a ordem física não for crescente por competência, o processamento **para cedo e ignora pagamentos válidos**. |
| 126 | Se o pagamento já estiver marcado como corrigido, então o sistema deve ignorá-lo. | Unwanted | `CALCCORR.NSN:L143-145` | Inferida | Garante idempotência. |
| 127 | O sistema deve corrigir o valor bruto multiplicando-o pelo índice IPCA do mês da competência. | Ubiquitous | `CALCCORR.NSN:L147-158, L191-206` | **Mistério** | 🚨🚨 A sub-rotina se chama `CALC-INDICE-ACUM`, mas aplica **um único mês** de IPCA — não acumula nada. Ver M-40. |
| 128 | Onde o ano da competência não constar na tabela de índices, o sistema deve manter o índice em 1,0. | Optional | `CALCCORR.NSN:L196-204` | **Mistério** | 🚨 A tabela tem espaço para 10 anos e apenas **3 estão carregados** (2010, 2011, 2012), embora o cabeçalho declare "ULTIMA CARGA: 2014". Correções de qualquer outro ano resultam em **zero**, silenciosamente. Ver M-41. |
| 129 | Se a diferença apurada for positiva, então o sistema deve gravar o valor corrigido, a data e o indicador de correção. | Unwanted | `CALCCORR.NSN:L160-170` | **Mistério** | Diferença negativa (deflação) nunca é aplicada. E o campo recebe o **valor corrigido total**, não a diferença, apesar do nome `VLR-CORRECAO`. |
| 130 | O sistema deve truncar o valor corrigido em centavos. | Ubiquitous | `CALCCORR.NSN:L153-156` | **Confirmada** | RN-014. |
| 131 | Ao final, o sistema deve exibir a quantidade de registros corrigidos e o valor total da correção. | Ubiquitous | `CALCCORR.NSN:L174-178` | Inferida | — |
| 132 | O sistema deve manter, comentado, o bloco de correção do Plano Verão (01/1989 a 01/1991). | Ubiquitous | `CALCCORR.NSN:L100-112` | **Mistério** | 🪦 Código morto preservado com fatores `2,7500` e `1,4289` e a instrução "NAO REMOVER (HISTORICO)". Nenhuma documentação explica os fatores. |

> 🐛 **A correção nunca chega ao valor pago.** O programa grava `VLR-CORRECAO` mas **não atualiza
> `VLR-LIQUIDO`**. O beneficiário não recebe a diferença. Além disso, os campos `VLR-CORRECAO`,
> `DT-CORRECAO` e `IND-CORRIGIDO` da VIEW (`L21-23`) **não existem em `PAGAMENTO.ddm`**.

> 🐛 **RN-019 não confere.** A documentação diz que o reajuste anual usa índice de decreto
> presidencial registrado no subprograma `CALCIDX`. Este programa usa **IPCA em tabela fixa**,
> e `CALCIDX` não existe no repositório.

## Regras de `BATCHREL.NSN`

**Programa:** `01-arqueologia/legado-sifap/natural-programs/BATCHREL.NSN` (~205 linhas) · Par 2 · Arquitetura
**Cabeçalho:** 1999 criação (Patrícia Gomes de Souza) · 2006 subtotais por região · 2013 ajuste de formato
**Acesso a dados:** lê `PAGAMENTO` (152) e `BENEFICIARIO` (150). **Somente leitura.**

| #   | Declaração da Regra | Candidata EARS | Fonte | Classificação | Observações |
| --- | --- | --- | --- | --- | --- |
| 133 | O sistema deve consolidar os pagamentos da competência informada por região, por situação e no total geral. | Ubiquitous | `BATCHREL.NSN:L103-172` | Inferida | — |
| 134 | O sistema deve agrupar as regiões pelas faixas 1-5 (Norte), 6-10 (Nordeste), 11-15 (Sudeste), 16-20 (Sul) e demais (Centro-Oeste). | Ubiquitous | `BATCHREL.NSN:L114-131` | **Mistério** | 🚨 **Quarta codificação regional do sistema.** A faixa "Sul" (16-20) inclui `MS` e `MT`, que são Centro-Oeste em `CALCBENF`. Ver M-34. |
| 135 | Onde a região do beneficiário não estiver entre 1 e 20, o sistema deve classificá-lo como Centro-Oeste. | Optional | `BATCHREL.NSN:L127-129` | **Mistério** | 🚨 O `ELSE` captura a **região 99** e também o valor **0** (beneficiário não encontrado). Ambos entram no consolidado como **Centro-Oeste**. Ver M-42. |
| 136 | O sistema deve **arredondar** o valor bruto ao acumular no relatório. | Ubiquitous | `BATCHREL.NSN:L133-140` | **Mistério** | 🚨🚨 O comentário da linha 134 admite: "ARREDONDAMENTO DIFERE DO CALCBENF (ROUND VS TRUNCATE)". O pagamento **trunca**, o relatório **arredonda**. Ver M-43. |
| 137 | O sistema deve acumular desconto e líquido sem arredondamento. | Ubiquitous | `BATCHREL.NSN:L141-142` | **Mistério** | Só o bruto é arredondado. No relatório, **bruto ≠ desconto + líquido**. |
| 138 | O sistema deve classificar as situações como Gerado (`G`), Pago (`P`), Cancelado (`C`), Devolvido (`D`) e Estornado (`E`). | Ubiquitous | `BATCHREL.NSN:L78-83, L145-159` | **Mistério** | 🚨🚨 Contradiz o DDM: `P=PENDENTE`, `C=CONFIRMADO`, `E=EMITIDO`. **Um pagamento confirmado é reportado como "CANCELADO".** Ver M-44. |
| 139 | Se a situação do pagamento não for reconhecida, então o sistema deve contabilizá-la como "Gerado". | Unwanted | `BATCHREL.NSN:L157-158` | Inferida | Situações `X` e `R` do DDM caem aqui e inflam o total de "gerados". |
| 140 | Ao final, o sistema deve imprimir os resumos por região, por situação e o total geral. | Ubiquitous | `BATCHREL.NSN:L174-200` | Inferida | A paginação (`#MAX-LINHAS`, `#LINHA`, `#PAG`) é declarada e o cabeçalho é impresso **uma única vez** — o controle de página nunca é testado no laço. `#LINHA-REL (A132)` nunca é usada. |

## Regras de `RELPGT.NSN`

**Programa:** `01-arqueologia/legado-sifap/natural-programs/RELPGT.NSN` (~215 linhas) · Par 5 · Operações
**Cabeçalho:** 1999 criação (Ana Lúcia Pereira) · 2004 ajuste de paginação · 2010 subtotal por programa
**Acesso a dados:** lê `PAGAMENTO` (152) e `BENEFICIARIO` (150). **Somente leitura.**

| #   | Declaração da Regra | Candidata EARS | Fonte | Classificação | Observações |
| --- | --- | --- | --- | --- | --- |
| 141 | O sistema deve listar os pagamentos do intervalo de competências informado. | Ubiquitous | `RELPGT.NSN:L81-85` | Inferida | — |
| 142 | Onde o código de programa for informado, o sistema deve filtrar apenas os pagamentos desse programa. | Optional | `RELPGT.NSN:L87-90` | Inferida | Filtro em memória, após a leitura. |
| 143 | Quando o código de programa mudar em relação ao registro anterior, o sistema deve imprimir o subtotal do programa. | Event-driven | `RELPGT.NSN:L92-99` | **Mistério** | 🚨 A leitura é ordenada por **competência**, mas a quebra é por **programa**. Como a ordem não acompanha a chave de quebra, o relatório gera **vários subtotais parciais do mesmo programa**. Ver M-45. |
| 144 | O sistema deve exibir o CPF mascarado no formato `***.NNN.NNN-NN`. | Ubiquitous | `RELPGT.NSN:L108-112` | **Mistério** | 🚨 Mascara apenas os **3 primeiros dígitos** e revela os **8 restantes** — muito mais fraco que a máscara de `CONSBENF` (regra 96). E este relatório é **impresso em papel**. Ver M-46. |
| 145 | O sistema deve traduzir o tipo de pagamento como Normal (`N`), Décimo (`D`) ou Terceiro (`T`). | Ubiquitous | `RELPGT.NSN:L114-124` | **Mistério** | O valor `'T'` **nunca é gravado** por nenhum programa — branch morto. E "Décimo" e "Terceiro" como tipos separados sugerem que `13º` foi partido em dois por engano. |
| 146 | O sistema deve traduzir a situação do pagamento como Gerado, Pago, Cancelado, Devolvido ou Estornado. | Ubiquitous | `RELPGT.NSN:L126-141` | **Mistério** | Mesma tradução incorreta de M-44. |
| 147 | O sistema deve imprimir cabeçalho a cada 61 linhas. | Ubiquitous | `RELPGT.NSN:L143-146, L180-192` | Inferida | Aqui a paginação **funciona**, ao contrário de `BATCHREL`. |
| 148 | Ao final, o sistema deve imprimir o último subtotal e os totais gerais, incluindo o total de abono. | Ubiquitous | `RELPGT.NSN:L165-178` | Inferida | ⚠️ **Não filtra por situação** — pagamentos cancelados e devolvidos entram nos totais gerais. |

## Regras de `RELAUDIT.NSN`

**Programa:** `01-arqueologia/legado-sifap/natural-programs/RELAUDIT.NSN` (~245 linhas) · Par 5 · Operações
**Cabeçalho:** 2002 criação (Roberto Mendes Junior) · 2006 inclusão de filtros · 2011 ajuste de formato · **2014 "LIMPEZA RELATORIO" (Fernanda Costa)**
**Acesso a dados:** lê `AUDITORIA` (153). **Somente leitura.**

| #   | Declaração da Regra | Candidata EARS | Fonte | Classificação | Observações |
| --- | --- | --- | --- | --- | --- |
| 149 | Onde a data inicial não for informada, o sistema deve assumir 01/01/1997; onde a final não for informada, deve assumir a data corrente. | Optional | `RELAUDIT.NSN:L82-87` | Inferida | 1997 é a data de criação do SIFAP. |
| 150 | O sistema deve listar os eventos de auditoria do período, ordenados por data. | Ubiquitous | `RELAUDIT.NSN:L90-98` | Inferida | Varredura completa dos ~25 milhões de registros a partir do início do arquivo. |
| 151 | **O sistema deve excluir da listagem todos os eventos de exclusão (`'EX'`), contabilizando-os apenas como "filtrados".** | Unwanted | `RELAUDIT.NSN:L100-108` | **Mistério** | 🚨🚨🚨 Ver M-47. O achado mais grave do Estágio 1. |
| 152 | Onde os filtros de ação, usuário ou tabela forem informados, o sistema deve restringir a listagem a eles. | Optional | `RELAUDIT.NSN:L110-133` | **Mistério** | 🚨 O filtro de exclusões roda **antes** destes. Solicitar explicitamente `ACAO = 'EX'` retorna **zero registros**, sem nenhuma mensagem. |
| 153 | O sistema deve classificar as ações como Inclusão (`IN`), Alteração (`AL`), Conciliação (`CO`), Consulta (`CN`) e Divergência (`DV`). | Ubiquitous | `RELAUDIT.NSN:L137-157` | **Mistério** | 🚨 No DDM, `CO = CONSULTA`. O relatório rotula todo `'CO'` como **"CONCILIAÇÃO"**, e inventa `'CN'` para consulta — código que **não existe no DDM**. Todo registro histórico de consulta é exibido com o rótulo errado. Ver M-30. |
| 154 | O sistema deve formatar a hora do evento como `HH:MM:SS`. | Ubiquitous | `RELAUDIT.NSN:L159-163` | Inferida | — |
| 155 | Onde a saída for impressora, o sistema deve incluir a descrição do evento; na tela, deve omiti-la. | Optional | `RELAUDIT.NSN:L170-186` | Inferida | A versão em tela esconde `DESCRICAO`, onde ficam os detalhes da divergência gravados por `BATCHCON`. |
| 156 | Ao final, o sistema deve exibir os totais por tipo de ação, incluindo a quantidade de registros filtrados. | Ubiquitous | `RELAUDIT.NSN:L190-208` | Inferida | O total de "filtrados" é a **única pista** de que exclusões foram ocultadas — sem explicar o motivo. |

> 🚨 **M-47 — A trilha de auditoria esconde as exclusões.**
>
> ```
> IF AUDITORIA-V.ACAO = 'EX'
>   ADD 1 TO #QTD-FILTRADOS
>   ESCAPE TOP
> END-IF
> ```
>
> O comentário acima do bloco é explícito: `FILTRO ACAO - EXCLUSOES NAO SAO EXIBIDAS`.
> A entrada de changelog correspondente é **"15/09/2014 — LIMPEZA RELATORIO"**.
>
> O DDM confirma de forma independente, em `AUDITORIA.ddm` (NOTA2):
> *"CUIDADO - PROGRAMA RELAUDIT.NSN FILTRA ACOES 'EX' NA EXIBICAO. PARA VER EXCLUSOES,
> CONSULTAR DIRETAMENTE VIA ADABAS ONLINE (SYSAOS)"*.
>
> O mesmo DDM declara que a retenção da trilha é **obrigação legal (IN-TCU 63/2010)** e que o
> arquivo é imutável. O dado **está gravado** — mas o único relatório que existe **não o mostra**.
> <!-- mystery: quem autorizou ocultar eventos de exclusão do relatório oficial de auditoria em 2014, e quais exclusões ocorreram desde então sem visibilidade -->

### Mistérios abertos do Lote C

| ID | Pergunta em aberto | Evidência | Impacto |
| --- | --- | --- | --- |
| M-34 | Qual é a codificação regional correta? Existem **quatro** incompatíveis entre si. | `CALCBENF.NSN:L87-119` (UF por posição, 3 estados faltando) · `BATCHPGT.NSN:L239-244` (1-25) · `BATCHREL.NSN:L114-131` (faixas de 5) · `BENEFICIARIO.ddm` `BJ` (01-05 ou 99) · `PROGRAMA-SOCIAL.ddm` `FA` (6 grupos) · RN-005 (01-27) | 🔴 **Crítico.** Bloqueia a modelagem de região no sistema-alvo. |
| M-35 | O 13º é proporcional aos meses ativos (comentário) ou valor cheio (código)? | `CALCBENF.NSN:L234-241`; `REGRAS-NEGOCIO-2012.md` §6 ("pro rata" — Alta prioridade) | 🔴 **Crítico.** É o cálculo pro rata que a doc declarou não documentado. |
| M-36 | Por que `CALCBENF` grava pagamento sem número e sem verificar duplicidade na competência? | `CALCBENF.NSN:L271-283` vs `BATCHPGT.NSN:L200-210, L323` | 🔴 **Crítico.** Pode gerar pagamentos duplicados e sem chave. |
| M-37 | A vigência do desconto deve ser avaliada na data de hoje ou na competência do pagamento? | `CALCDSCT.NSN:L111-118` | 🟠 Alto. Reprocessamento aplica descontos errados. |
| M-38 | Quando o teto de 30% é atingido, o correto é truncar o total (código) ou descartar descontos por prioridade (RN-023)? | `CALCDSCT.NSN:L179-184`; RN-023 | 🔴 **Crítico.** No código, um desconto judicial já somado pode ser apagado por um desconto comum posterior. |
| M-39 | Por que `CALCDSCT` e `CALCCORR` atualizam o pagamento sem recalcular o valor líquido? | `CALCDSCT.NSN:L191-196`; `CALCCORR.NSN:L160-170` | 🔴 **Crítico.** O valor efetivamente pago fica inconsistente com bruto e desconto. |
| M-40 | Por que a rotina chamada `CALC-INDICE-ACUM` aplica um único mês de índice em vez de acumular o período? | `CALCCORR.NSN:L191-206` | 🔴 **Crítico.** A correção retroativa não corrige retroativamente. |
| M-41 | Por que a tabela de IPCA só tem 2010–2012 se o cabeçalho declara carga até 2014, e o programa foi alterado em 2014? | `CALCCORR.NSN:L47-99` | 🟠 Alto. Correções fora desses 3 anos resultam em zero, sem aviso. |
| M-42 | Beneficiários da região 99 e beneficiários não localizados são somados ao Centro-Oeste. É intencional? | `BATCHREL.NSN:L127-129` | 🟠 Alto. Distorce o consolidado usado para prestação de contas. |
| M-43 | O relatório deve arredondar (código do relatório) ou truncar (código do pagamento e RN-014)? | `BATCHREL.NSN:L133-140` e comentário `L134` | 🔴 **Crítico.** Os totais do relatório **não fecham** com a soma dos pagamentos. Divergência conhecida e documentada no próprio código. |
| M-44 | Qual é o significado real das situações de pagamento `P`, `C` e `E`? | `PAGAMENTO.ddm` campo `DA` vs `BATCHREL.NSN:L78-83` vs `RELPGT.NSN:L126-141` vs `BATCHCON.NSN:L174-196` | 🔴 **Crítico.** Três interpretações diferentes no mesmo sistema. Um pagamento confirmado é impresso como "CANCELADO". |
| M-45 | Por que a quebra de subtotal é por programa se a leitura é ordenada por competência? | `RELPGT.NSN:L81, L92-99` | 🟠 Alto. Subtotais fragmentados e não somáveis. |
| M-46 | Por que existem duas máscaras de CPF diferentes, e por que a mais fraca é usada no relatório impresso? | `RELPGT.NSN:L108-112` vs `CONSBENF.NSN:L175-190` | 🔴 **Crítico.** Exposição de dado pessoal em papel. |
| M-47 | Quem autorizou ocultar os eventos de exclusão do relatório oficial de auditoria? | `RELAUDIT.NSN:L100-108`; cabeçalho "2014 — LIMPEZA RELATORIO"; `AUDITORIA.ddm` NOTA2; IN-TCU 63/2010 | 🔴 **Crítico — o mais grave do Estágio 1.** Descumprimento potencial de obrigação legal de trilha de auditoria. |

> 💡 Duplique a seção acima para cada programa `.NSN` lido pelo seu par.

## Resumo Geral

**Cobertura: 15 de 15 programas Natural e 4 de 4 DDMs. Estágio 1 completo.**

| Métrica | Valor |
| --- | ---: |
| Programas Natural lidos | **15 de 15** |
| DDMs cruzados | **4 de 4** |
| Documentos históricos cruzados | 1 de 3 (`REGRAS-NEGOCIO-2012.md`) |
| Blocos condicionais examinados | 100% dos 15 programas |
| **Total de regras candidatas** | **156** |
| Regras Confirmadas | 23 |
| Regras Inferidas | 76 |
| Regras classificadas como Mistério | 57 |
| **Perguntas em aberto registradas** | **47** (`M-01` … `M-47`) |
| Das quais críticas 🔴 | **23** |

### Cobertura por programa

| Par | Programa | Regras | Confirmadas | Inferidas | Mistérios |
| --- | --- | ---: | ---: | ---: | ---: |
| 1 · Visão | `CADBENEF.NSN` | 12 | 5 | 5 | 2 |
| 1 · Visão | `CADDEPEND.NSN` | 8 | 0 | 6 | 2 |
| 1 · Visão | `CADPROG.NSN` | 5 | 0 | 4 | 1 |
| 2 · Arquitetura | `BATCHPGT.NSN` | 20 | 4 | 8 | 8 |
| 2 · Arquitetura | `BATCHREL.NSN` | 8 | 0 | 3 | 5 |
| 2 · Arquitetura | `BATCHCON.NSN` | 12 | 0 | 7 | 5 |
| 3 · Implementação | `CALCBENF.NSN` | 12 | 3 | 3 | 6 |
| 3 · Implementação | `CALCCORR.NSN` | 9 | 1 | 3 | 5 |
| 3 · Implementação | `CALCDSCT.NSN` | 12 | 2 | 6 | 4 |
| 4 · Qualidade | `VALBENEF.NSN` | 9 | 2 | 5 | 2 |
| 4 · Qualidade | `VALDOCS.NSN` | 4 | 1 | 2 | 1 |
| 4 · Qualidade | `VALELEG.NSN` | 20 | 4 | 10 | 6 |
| 5 · Operações | `CONSBENF.NSN` | 9 | 1 | 5 | 3 |
| 5 · Operações | `RELPGT.NSN` | 8 | 0 | 4 | 4 |
| 5 · Operações | `RELAUDIT.NSN` | 8 | 0 | 5 | 3 |
| | **Total** | **156** | **23** | **76** | **57** |

### Os 6 mistérios que devem abrir a Passagem #1

Estes bloqueiam decisão de escopo. Nenhum pode ser resolvido pelo agente — todos precisam de validação humana.

| ID | Assunto | Por que bloqueia |
| --- | --- | --- |
| **M-47** | Relatório de auditoria oculta exclusões | Possível descumprimento de obrigação legal (IN-TCU 63/2010). Escalar antes de qualquer decisão técnica. |
| **M-14** | Suspensão automática acima de 75 anos | Remove o benefício de todo idoso, sem base documental. |
| **M-44** | Significado real das situações `P`, `C` e `E` | Três interpretações no mesmo sistema; um pagamento confirmado é reportado como cancelado. |
| **M-34** | Quatro codificações regionais incompatíveis | Bloqueia a modelagem de região no schema-alvo. |
| **M-20 / M-24** | Cálculo e reajuste duplicados | O batch que paga 4,2 milhões de pessoas usa a própria cópia da fórmula; o reajuste é aplicado duas vezes. |
| **M-18 / M-19** | Backdoors de CPF | `000.000.000-00` é válido, e 8 prefixos anulam toda a validação documental. |

### O que ainda falta no Estágio 1

| Artefato | Estado |
| --- | --- |
| [`mysteries-found.md`](mysteries-found.md) | ❌ Os 47 mistérios precisam ser formalizados com responsável e status → `/catalog-mysteries` |
| [`dependency-map.md`](dependency-map.md) | ❌ Vazio → `/map-dependencies`. Atenção: o call graph tem **zero arestas** (E-01); o mapa real é de acesso a dados |
| [`glossary.md`](glossary.md) | ❌ 0 de 15 termos |
| [`discovery-report.md`](discovery-report.md) | ❌ Vazio → `/discovery-report` |
| Documentação histórica | 🟡 `MANUAL-TECNICO-SIFAP-2008.md` e `ARQUITETURA-ORIGINAL-1997.md` ainda não cruzados |

---

✅ **Critério de pronto:** todo bloco condicional dos programas atribuídos examinado, cada regra cita `arquivo:linha`, e toda pergunta em aberto é registrada em `mysteries-found.md` sem conclusão pelo agente.

---

### Continuar a leitura

<table width="100%">
<tr>
<td width="50%" valign="top" align="left">
<sub><strong>← ANTERIOR</strong></sub><br/>
<a href="inventory.md"><strong>Inventário</strong></a><br/>
<sub>Passo 1.</sub>
</td>
<td width="50%" valign="top" align="right">
<sub><strong>PRÓXIMO →</strong></sub><br/>
<a href="dependency-map.md"><strong>Mapa de Dependências</strong></a><br/>
<sub>Passo 3.</sub>
</td>
</tr>
</table>

<sub>↑ <a href="../README.md">Voltar ao Kit PT-BR</a></sub>

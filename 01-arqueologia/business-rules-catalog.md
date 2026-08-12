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

> 💡 Duplique a seção acima para cada programa `.NSN` lido pelo seu par.

## Resumo Geral

| Métrica | Valor |
| --- | ---: |
| Programas Natural lidos | 1 de 15 (`VALELEG.NSN`) |
| DDMs cruzados | 1 de 4 (`BENEFICIARIO.ddm`) |
| Documentos históricos cruzados | 1 de 3 (`REGRAS-NEGOCIO-2012.md`) |
| Blocos condicionais examinados | 20 (`IF`/`DECIDE`/`FOR`) — 100% do programa |
| Regras Confirmadas | 4 |
| Regras Inferidas | 10 |
| Mistérios (regra existe, intenção incerta) | 6 |
| Mistérios (regra documentada, código ausente) | 4 |
| **Total de regras candidatas** | **20** |

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

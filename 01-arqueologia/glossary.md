<!-- markdownlint-disable MD012 MD013 MD022 MD025 MD026 MD028 MD029 MD031 MD033 MD034 MD038 MD040 MD051 MD060 -->

# Glossário do SIFAP Legado

![ESTÁGIO 01 Arqueologia](https://img.shields.io/badge/ESTÁGIO-01%20Arqueologia-F25022?style=for-the-badge) ![TIPO Worksheet](https://img.shields.io/badge/TIPO-Worksheet-1A1A1A?style=for-the-badge) ![PREENCHA Durante S1](https://img.shields.io/badge/PREENCHA-Durante%20S1-737373?style=for-the-badge)

> 🗺 **Você está aqui:** [Kit PT-BR](../README.md) → [Estágio 1](README.md) → **glossary**

> **Para quem é isto?** Este é um **artefato preenchido pelo time** durante o Estágio 1 (Arqueologia).
>
> **O que você terá ao final do estágio:**
>
> 1. Tabela com todos os termos, abreviações e siglas do código Natural/Adabas
> 2. Vocabulário comum entre os pares — base da linguagem ubíqua do Estágio 2
> 3. Base de evidência usada nas EARS do Estágio 2 (`source_legacy:`)
>
> 📘 **Guia passo a passo:** [`GUIDE.md`](GUIDE.md).


> Preencha esta tabela com todos os termos, abreviações e siglas encontrados no código Natural/Adabas.
> **Meta: no mínimo 30 termos.**

## Por que isso importa

Sistemas legados têm vocabulário próprio que ninguém documenta em lugar nenhum — só está no nome das variáveis. Se o time do Estágio 2 não souber o que `DSCT`, `BENF`, `PE` ou `CTC` significam, vai escrever uma spec sobre o que ele _acha_ que isso significa. Glossário é o que evita esse desencontro.

## Como preencher

- **Termo**: a abreviação ou sigla exatamente como aparece no código
- **Expansão**: o significado completo do termo
- **Programa**: em qual arquivo `.NSN` ou `.ddm` o termo foi encontrado
- **Contexto**: breve explicação de como/onde o termo é usado
- **Status**: **CONFIRMADO** quando há evidência literal no código/doc; **HIPÓTESE** quando inferido pelo contexto e ainda precisa de validação

## Dica de extração

Prompt útil no Copilot Chat (cole o conteúdo de 2–3 arquivos `.NSN` no chat antes):

> _"Liste todas as abreviações e siglas usadas neste código Natural. Para cada uma, sugira a expansão e marque com 'CONFIRMADO' ou 'HIPÓTESE'."_

## Termos encontrados

**Origem:** varredura completa do Estágio 1 — 15 programas `.NSN`, 4 DDMs e `REGRAS-NEGOCIO-2012.md`.
**Critério de status:** `CONFIRMADO` = a expansão aparece **literalmente** no código ou na documentação. `HIPÓTESE` = deduzida do contexto e ainda **não validada**.
**Total: 113 termos — 85 CONFIRMADO, 28 HIPÓTESE.**

### A · Sistema e entidades de domínio

| #  | Termo | Expansão | Programa | Contexto | Status |
| -- | ----- | -------- | -------- | -------- | ------ |
| 1 | `SIFAP` | Sistema de Fiscalização e Administração de Pagamentos | cabeçalho dos 15 `.NSN` e dos 4 `.ddm` | Nome do sistema. Ex.: `VALELEG.NSN:L3` | CONFIRMADO |
| 2 | `BENEFICIARIO` | Cadastro de beneficiários de programas sociais | `BENEFICIARIO.ddm:L9` | Arquivo principal. ~4,2 milhões de registros | CONFIRMADO |
| 3 | `PROGRAMA-SOCIAL` | Cadastro de programas sociais e regras de elegibilidade | `PROGRAMA-SOCIAL.ddm:L9` | Tabela paramétrica. ~45 programas ativos | CONFIRMADO |
| 4 | `PAGAMENTO` | Histórico de pagamentos processados pelo SIFAP | `PAGAMENTO.ddm:L10` | Tabela transacional. ~180 milhões de registros | CONFIRMADO |
| 5 | `AUDITORIA` | Log de auditoria — trilha de alterações do SIFAP | `AUDITORIA.ddm:L9` | Registro imutável. ~25 milhões de registros | CONFIRMADO |
| 6 | `CPF` | Cadastro de Pessoas Físicas | todos os programas | Chave de negócio do beneficiário. O DDM registra apenas "CPF SEM FORMATACAO" (`BENEFICIARIO.ddm:L23`) | HIPÓTESE |
| 7 | `NIS` | Número de Identificação Social | `VALELEG.NSN:L24`; `CONSBENF.NSN:L92` | Usado como chave alternativa de busca. **Não existe no DDM** (ver M-06) | HIPÓTESE |
| 8 | `NIT` | Número de Identificação do Trabalhador | `REGRAS-NEGOCIO-2012.md` RN-001 | Citado como "NIS/NIT" sem expansão | HIPÓTESE |
| 9 | `CTPS` | Carteira de Trabalho e Previdência Social | `VALDOCS.NSN:L23` | Campo de entrada declarado e nunca validado | HIPÓTESE |
| 10 | `RG` | Registro Geral (documento de identidade) | `BENEFICIARIO.ddm:L27-30`; `VALDOCS.NSN` | Grupo de 4 campos: número, órgão, UF e data | HIPÓTESE |
| 11 | `Dependente` | Pessoa vinculada ao beneficiário titular | `CADDEPEND.NSN:L7`; `BENEFICIARIO.ddm:L60` | Grupo periódico com até 10 ocorrências. Limite real em disputa (M-15) | CONFIRMADO |
| 12 | `Titular` | Beneficiário ao qual dependentes são vinculados | `CADDEPEND.NSN:L8, L25` | Termo usado apenas em `CADDEPEND` | CONFIRMADO |
| 13 | `Competência` | Mês de referência do pagamento, no formato AAAAMM | `PAGAMENTO.ddm:L26` (`AE ANO-MES-REF -- AAAAMM - COMPETENCIA`) | Chave temporal de todo o fluxo financeiro | CONFIRMADO |
| 14 | `Ciclo` | Ciclo de processamento | `PAGAMENTO.ddm:L27` (`AF NUM-CICLO -- CICLO PROCESSAMENTO`) | Campo do DDM. Nenhum programa o grava | CONFIRMADO |
| 15 | `Elegibilidade` | Verificação de requisitos do cadastro contra o programa | `VALELEG.NSN:L9-10` | Núcleo de `VALELEG`. Não é consultado pelo batch de pagamento | CONFIRMADO |
| 16 | `Renda per capita` | Renda familiar dividida pelos membros do domicílio | `BENEFICIARIO.ddm:L57` (`CJ IND-RENDA-PERCAP -- RENDA PER CAPITA CALC`) | Campo calculado. **Nunca lido por nenhum programa** (M-02) | CONFIRMADO |
| 17 | `Faixa de renda` | Intervalo de renda que determina o fator multiplicador | `CALCBENF.NSN:L121-132`; `PROGRAMA-SOCIAL.ddm:L61` | 5 faixas fixas em código; o DDM prevê grupo periódico de faixas | CONFIRMADO |
| 18 | `Abono natalino` | Adicional de 15% em dezembro para programas assistenciais | `CALCBENF.NSN:L7, L248` | Percentual fixo em código, sem documentação | CONFIRMADO |
| 19 | `13º` / `Décimo` | Parcela adicional calculada na competência de dezembro | `CALCBENF.NSN:L6, L233` | Fórmula do comentário difere da implementada (M-35) | CONFIRMADO |
| 20 | `Pro rata` | Cálculo proporcional para benefício iniciado no meio do mês | `REGRAS-NEGOCIO-2012.md` §2.1 nota e §6 | Declarado como não documentado e não implementado | CONFIRMADO |
| 21 | `Conciliação` | Confronto entre pagamentos do SIFAP e retorno bancário | `BATCHCON.NSN:L8` | Processo do `BATCHCON` sobre arquivo CNAB 240 | CONFIRMADO |
| 22 | `FATOR-K` | Fator de correção especial | `PROGRAMA-SOCIAL.ddm:L43` (`BG FATOR-K -- FATOR CORRECAO ESPECIAL`) | Campo marcado ">>> NAO DOCUMENTADO <<<". Fórmula localizada em `CADPROG.NSN:L87` (M-17) | CONFIRMADO |
| 23 | `Fator regional` | Multiplicador aplicado conforme o código de região | `CALCBENF.NSN:L88-119` | Tabela de 27 posições, 25 alcançáveis, mapeamento inconsistente (M-34) | CONFIRMADO |
| 24 | `Plano Verão` | Correção monetária da transição Cruzado → Cruzeiro (1989-1991) | `CALCCORR.NSN:L102-103` | Bloco comentado com instrução "NAO REMOVER (HISTORICO)" | CONFIRMADO |
| 25 | `IPCA` | Índice Nacional de Preços ao Consumidor Amplo | `CALCCORR.NSN:L6, L9` | Tabela fixa em código, apenas 2010–2012 (M-41) | HIPÓTESE |
| 26 | `CadÚnico` | Cadastro Único para Programas Sociais | `REGRAS-NEGOCIO-2012.md` RN-016 | Integração de 2006; programa não localizado | HIPÓTESE |

### B · Prefixos de nome de programa

| #  | Termo | Expansão | Programa | Contexto | Status |
| -- | ----- | -------- | -------- | -------- | ------ |
| 27 | `CAD` | Cadastro | `CADBENEF`, `CADDEPEND`, `CADPROG` | `CADBENEF.NSN:L8` declara "CADASTRO DE BENEFICIARIO" | CONFIRMADO |
| 28 | `VAL` | Validação | `VALBENEF`, `VALDOCS`, `VALELEG` | `VALELEG.NSN:L9` declara "VALIDACAO ELEGIBILIDADE" | CONFIRMADO |
| 29 | `CALC` | Cálculo | `CALCBENF`, `CALCCORR`, `CALCDSCT` | `CALCBENF.NSN:L10` declara "CALCULO VALOR BENEFICIO MENSAL" | CONFIRMADO |
| 30 | `REL` | Relatório | `RELPGT`, `RELAUDIT`, `BATCHREL` | `RELPGT.NSN:L7` declara "RELATORIO ANALITICO DE PAGAMENTOS" | CONFIRMADO |
| 31 | `CONS` | Consulta | `CONSBENF` | `CONSBENF.NSN:L9` declara "CONSULTA DADOS BENEFICIARIO" | CONFIRMADO |
| 32 | `BATCH` | Processamento em lote | `BATCHPGT`, `BATCHCON`, `BATCHREL` | `BATCHPGT.NSN:L12` declara "BATCH CRITICO - EXECUCAO 1O DIA UTIL" | CONFIRMADO |
| 33 | `BENEF` / `BENF` | Beneficiário | `CADBENEF`, `VALBENEF`, `CONSBENF` / `CALCBENF` | Duas grafias para a mesma entidade — inconsistência sinalizada no inventário | CONFIRMADO |
| 34 | `PGT` / `PGTO` | Pagamento | `BATCHPGT`, `RELPGT`; `NUM-PAGTO` | Duas grafias, ambas em uso | CONFIRMADO |
| 35 | `DSCT` | Desconto | `CALCDSCT`; `VLR-DESCONTO` | `CALCDSCT.NSN:L8` declara "CALCULO DESCONTOS E DEDUCOES" | CONFIRMADO |
| 36 | `CORR` | Correção | `CALCCORR` | `CALCCORR.NSN:L8` declara "CALCULO CORRECAO RETROATIVA" | CONFIRMADO |
| 37 | `ELEG` | Elegibilidade | `VALELEG`; `COD-ELEGIBILIDADE` | `VALELEG.NSN:L9` | CONFIRMADO |
| 38 | `DEPEND` | Dependente | `CADDEPEND`; `NUM-DEPENDENTES` | `CADDEPEND.NSN:L7` | CONFIRMADO |
| 39 | `PROG` | Programa | `CADPROG`; `COD-PROGRAMA` | `CADPROG.NSN:L8` declara "CADASTRO PROGRAMAS SOCIAIS" | CONFIRMADO |
| 40 | `AUDIT` | Auditoria | `RELAUDIT`; `SEQ-AUDIT` | `RELAUDIT.NSN:L8` declara "RELATORIO TRILHA AUDITORIA" | CONFIRMADO |
| 41 | `DOCS` | Documentos | `VALDOCS` | `VALDOCS.NSN:L8` declara "VALIDACAO DE DOCUMENTOS" | CONFIRMADO |
| 42 | `CON` | Conciliação | `BATCHCON` | `BATCHCON.NSN:L8` declara "CONCILIACAO PAGAMENTOS" | CONFIRMADO |

### C · Prefixos de nome de campo

| #  | Termo | Expansão | Programa | Contexto | Status |
| -- | ----- | -------- | -------- | -------- | ------ |
| 43 | `VLR` | Valor | todos os DDMs | `PAGAMENTO.ddm:L31` (`BA VLR-BRUTO -- VALOR BRUTO CALCULADO`) | CONFIRMADO |
| 44 | `QTD` | Quantidade | `BENEFICIARIO.ddm:L55` | `CI QTD-MEMBROS-FAMILIA -- MEMBROS NO DOMICILIO` | CONFIRMADO |
| 45 | `DT` | Data | todos os DDMs | Sempre no formato `AAAAMMDD`. Ex.: `BENEFICIARIO.ddm:L25` | CONFIRMADO |
| 46 | `HR` | Hora | `AUDITORIA.ddm:L23` | Formato `HHMMSS` | CONFIRMADO |
| 47 | `TS` | Timestamp | `AUDITORIA.ddm:L24` | `AD TS-EVENTO -- AAAAMMDDHHMMSS (PRECISAO)` | HIPÓTESE |
| 48 | `NUM` | Número | todos os DDMs | Ex.: `PAGAMENTO.ddm:L22` (`AA NUM-PAGAMENTO -- SEQUENCIAL UNICO`) | CONFIRMADO |
| 49 | `COD` | Código | todos os DDMs | Ex.: `PAGAMENTO.ddm:L64` (`EA COD-BANCO -- COD FEBRABAN`) | CONFIRMADO |
| 50 | `SIT` | Situação | `BENEFICIARIO.ddm:L52`; `PAGAMENTO.ddm:L45` | `CE SIT-BENEFICIARIO`, `DA SIT-PAGAMENTO` | CONFIRMADO |
| 51 | `IND` | Indicador | `BENEFICIARIO.ddm:L57, L79` | Campos booleanos com valores `S`/`N`. Expansão não literal | HIPÓTESE |
| 52 | `DES` | Descrição | `AUDITORIA.ddm:L37` | `BC DES-ACAO -- DESCRICAO LIVRE DA ACAO` | CONFIRMADO |
| 53 | `USR` | Usuário | `BENEFICIARIO.ddm:L86` | `GC USR-INCLUSAO -- LOGIN NATURAL` | CONFIRMADO |
| 54 | `MOT` | Motivo | `BENEFICIARIO.ddm:L53` | `CF MOT-SITUACAO -- COD MOTIVO (TAB INTERNA)` | CONFIRMADO |
| 55 | `PCT` | Percentual | `PAGAMENTO.ddm:L40` | `CD PCT-DESCONTO -- PERCENTUAL APLICADO` | CONFIRMADO |
| 56 | `SEQ` | Sequencial | `BATCHCON.NSN:L30`; `PAGAMENTO.ddm:L22` | `NUM-PAGAMENTO -- SEQUENCIAL UNICO` | CONFIRMADO |
| 57 | `ANT` | Anterior | `AUDITORIA.ddm:L48`; `BATCHPGT.NSN:L91` | `DC VALOR-ANTERIOR`, `#CPF-ANT` | CONFIRMADO |
| 58 | `ULT` | Último | `BENEFICIARIO.ddm:L88` | `GD DT-ULT-ALTERACAO` | CONFIRMADO |
| 59 | `APLIC` | Aplicável | `PROGRAMA-SOCIAL.ddm:L72` | `EA TIPO-DSCT-APLIC -- TIPOS DESCONTO VALIDOS` | CONFIRMADO |
| 60 | `PERCAP` | Per capita | `BENEFICIARIO.ddm:L57`; `PROGRAMA-SOCIAL.ddm:L50` | `RENDA-MAX-PERCAP -- RENDA PER CAPITA MAXIMA` | CONFIRMADO |
| 61 | `GRP` | Grupo | `BENEFICIARIO.ddm:L34, L60` | Prefixo de grupo de campos e de grupo periódico | CONFIRMADO |
| 62 | `REF` | Referência | `BATCHCON.NSN:L35-36` | `TABELA-REF`, `CHAVE-REF`. Também aparece como rótulo isolado em `CALCBENF.NSN:L102` | HIPÓTESE |

### D · Códigos de domínio (valores)

| #  | Termo | Expansão | Programa | Contexto | Status |
| -- | ----- | -------- | -------- | -------- | ------ |
| 63 | Situação do beneficiário: `A` `S` `C` `I` `D` | Ativo · Suspenso · Cancelado · Inativo · Desligado | `BENEFICIARIO.ddm:L52` | `CE SIT-BENEFICIARIO -- A=ATV S=SUSP C=CANC I=INAT D=DESL`. Confirmado também em `VALBENEF.NSN:L165-171` e `CONSBENF.NSN:L110-123` | CONFIRMADO |
| 64 | Situação do beneficiário: `E` | Excluído logicamente | `REGRAS-NEGOCIO-2012.md` RN-002, RN-011 | **Não existe no DDM nem em nenhum programa** (M-52) | HIPÓTESE |
| 65 | Situação do pagamento: `P` `G` `E` `C` `D` `X` `R` | Pendente · Gerado · Emitido · Confirmado · Devolvido · Cancelado · Reprocessado | `PAGAMENTO.ddm:L45-47` | ⚠️ Os relatórios usam significados divergentes para `P`, `C` e `E` (M-44) | CONFIRMADO |
| 66 | Tipo de programa: `A` `T` `P` | Assistencial · Trabalho · Previdenciário | `PROGRAMA-SOCIAL.ddm:L25` | `AD TIPO-PROGRAMA -- A=ASSISTENC T=TRABALHO P=PREVID` | CONFIRMADO |
| 67 | Situação do programa: `A` `I` `E` | Ativo · Inativo · Encerrado | `PROGRAMA-SOCIAL.ddm:L30` | `AI SIT-PROGRAMA -- A=ATV I=INAT E=ENCERR` | CONFIRMADO |
| 68 | Tipo de desconto (DDM): `IR` `JD` `CS` `PA` `EM` `TX` `OU` `EX` | IRRF · Judicial · Consignado · Pensão alimentícia · Empréstimo · Taxa · Outros · Extraordinário | `PROGRAMA-SOCIAL.ddm:L73-76` | ⚠️ `CALCDSCT.NSN:L26` usa um conjunto **incompatível** de 1 letra (M-38, taxonomias divergentes) | CONFIRMADO |
| 69 | Tipo de desconto (código): `C` `I` `J` `S` `P` `A` | Contribuição · Imposto · Judicial · Sindical · Pensão · Administrativo | `CALCDSCT.NSN:L26-27` | Comentário literal na declaração da VIEW | CONFIRMADO |
| 70 | Ação de auditoria: `IN` `AL` `EX` `CO` `LG` `LO` `BT` `ER` `AU` `RE` | Inclusão · Alteração · Exclusão · Consulta · Login · Logout · Batch · Erro · Autorização · Rejeição | `AUDITORIA.ddm:L27-36` | ⚠️ `BATCHCON` grava `CO` como "conciliação" e `DV`, ausente da lista (M-30) | CONFIRMADO |
| 71 | Parentesco (DDM): `FI` `CJ` `NT` `TU` | Filho · Cônjuge · Neto · Tutelado | `BENEFICIARIO.ddm:L64` | ⚠️ `CADDEPEND.NSN:L20` usa `FI` `CO` `IR` `OU` — conjunto divergente (M-16) | CONFIRMADO |
| 72 | Perfil de usuário: `ADM` `OPR` `CON` `AUD` `SUP` | Administrador · Operador · Consulta · Auditor · Supervisor | `AUDITORIA.ddm:L57` | `EC COD-PERFIL`. Nenhum programa lê este campo | CONFIRMADO |
| 73 | Tipo de entidade: `BENF` `PGTO` `PROG` `ADMN` `SIST` | Beneficiário · Pagamento · Programa · Administrativo · Sistema | `AUDITORIA.ddm:L41` | `CA TIPO-ENTIDADE` | CONFIRMADO |

### E · Termos técnicos Natural / Adabas

| #  | Termo | Expansão | Programa | Contexto | Status |
| -- | ----- | -------- | -------- | -------- | ------ |
| 74 | `DDM` | Data Definition Module | os 4 arquivos `.ddm` | Visão Natural sobre um arquivo Adabas. Sigla não expandida nos arquivos | HIPÓTESE |
| 75 | `FNR` | File Number | cabeçalho dos 4 `.ddm` | Identificador numérico do arquivo Adabas. ⚠️ Os programas citam números divergentes (M-48) | HIPÓTESE |
| 76 | `DBID` | Database Identifier | cabeçalho dos 4 `.ddm` | Todos os arquivos usam `DBID: 57` | HIPÓTESE |
| 77 | `DE` | Descriptor — campo indexado para busca | `BENEFICIARIO.ddm:L96` | `(DE) = DESCRIPTOR (CAMPO INDEXADO PARA BUSCA)` | CONFIRMADO |
| 78 | `PE` | Periodic Group — grupo de campos repetitivo | `BENEFICIARIO.ddm:L97` | `(PE) = PERIODIC GROUP`. Usado para dependentes (máx. 10) e descontos (máx. 8) | CONFIRMADO |
| 79 | `MU` | Multiple Value Field — campo multivalorado | `BENEFICIARIO.ddm:L98` | `(MU) = MULTIPLE VALUE FIELD` | CONFIRMADO |
| 80 | `Superdescriptor` | Chave composta por trechos de vários campos | `BENEFICIARIO.ddm:L100-103` | Ex.: `S3 = CA(1-4) + CE(1-1) -- PROGRAMA + SITUACAO` | CONFIRMADO |
| 81 | `ISN` | Internal Sequence Number | `BENEFICIARIO.ddm:L22` | `AA NUM-INSCRICAO -- ISN ALTERNATIVO / MATRICULA` | HIPÓTESE |
| 82 | `OCC` | Ocorrências de um grupo periódico | cabeçalho de coluna dos 4 `.ddm` | Coluna que declara o máximo de repetições | CONFIRMADO |
| 83 | Formatos `A` `N` `P` `B` | Alfanumérico · Numérico · Packed decimal · Binário | coluna `FORMAT` dos 4 `.ddm` | Apenas `A` e `N` são usados no SIFAP | HIPÓTESE |
| 84 | `VIEW OF` | Projeção de campos de um DDM dentro de um programa | os 15 `.NSN` | ⚠️ As VIEWs divergem dos DDMs em nome e tipo (M-12) | CONFIRMADO |
| 85 | `WORK FILE` | Arquivo sequencial externo ao banco | `BATCHCON.NSN:L105-106` | Único ponto de entrada de dados externos do sistema (CNAB 240) | CONFIRMADO |
| 86 | `MAP` | Definição de layout de tela de terminal | `CONSBENF.NSN:L69` | Referencia `CONSBENF-M01`. **Nenhum arquivo `.map` existe no repositório** (M-31) | CONFIRMADO |
| 87 | `PERFORM` | Chamada de sub-rotina interna ao próprio programa | 23 ocorrências nos 15 `.NSN` | Único mecanismo de reúso presente. Não há `CALLNAT` (M-49) | CONFIRMADO |
| 88 | `*DATN` / `*TIMN` | Variáveis de sistema: data e hora numéricas | `VALELEG.NSN:L59`; `BATCHCON.NSN:L83` | Fonte de data em todos os programas | HIPÓTESE |
| 89 | `*NUMBER` | Variável de sistema: quantidade de registros do último acesso | `CADPROG.NSN:L121`; `CALCDSCT.NSN:L91` | Idioma correto; os demais programas usam flag manual | HIPÓTESE |
| 90 | `*ERROR-NR` | Variável de sistema: número do último erro | `CONSBENF.NSN:L72` | Usada para detectar falha na carga do mapa | HIPÓTESE |

### F · Siglas organizacionais e normativas

| #  | Termo | Expansão | Programa | Contexto | Status |
| -- | ----- | -------- | -------- | -------- | ------ |
| 91 | `SENARC` | Secretaria Nacional de Renda de Cidadania | `REGRAS-NEGOCIO-2012.md` cabeçalho | Área de negócio responsável pelas regras | CONFIRMADO |
| 92 | `CGPB` | Coordenação-Geral de Pagamento de Benefícios | `REGRAS-NEGOCIO-2012.md` distribuição; RN-004 | Autoriza exceções ao limite de dependentes | HIPÓTESE |
| 93 | `CGTI` | Coordenação-Geral de Tecnologia da Informação | `REGRAS-NEGOCIO-2012.md`; `AUDITORIA.ddm` nota | Autor da Portaria 213/2010 que suspendeu o log de consultas | HIPÓTESE |
| 94 | `MDAS` / `MDS` | Ministério do Desenvolvimento e Assistência Social | `PROGRAMA-SOCIAL.ddm:L26`; `BENEFICIARIO.ddm:L11` | `AE ORGAO-RESPONSAVEL -- COD ORGAO MDS/MDAS` | HIPÓTESE |
| 95 | `SUPDE` / `DESIF` | Unidades citadas na distribuição do documento | `REGRAS-NEGOCIO-2012.md` cabeçalho | Sem expansão em nenhuma fonte | HIPÓTESE |
| 96 | `DEGED` | Unidade de destino de transferência de pessoal | `REGRAS-NEGOCIO-2012.md` §6 | Destino da responsável pelas regras de conciliação | HIPÓTESE |
| 97 | `CNAB 240` | Layout de arquivo bancário de 240 posições | `BATCHCON.NSN:L9` | "PROCESSO CNAB 240 BANCO DO BRASIL" | CONFIRMADO |
| 98 | `SIAFI` | Sistema Integrado de Administração Financeira | `PAGAMENTO.ddm:L70` | Grupo de 5 campos. Nenhum programa os grava | HIPÓTESE |
| 99 | `OB` | Ordem Bancária | `PAGAMENTO.ddm:L71` | `FA NUM-OB-SIAFI -- ORDEM BANCARIA SIAFI` | CONFIRMADO |
| 100 | `NE` | Nota de Empenho | `PAGAMENTO.ddm:L72` | `FB NUM-NE-SIAFI -- NOTA EMPENHO SIAFI` | CONFIRMADO |
| 101 | `UG` | Unidade Gestora | `PAGAMENTO.ddm:L73` | `FC COD-UG-EMITENTE -- UNIDADE GESTORA` | CONFIRMADO |
| 102 | `FEBRABAN` | Federação Brasileira de Bancos | `PAGAMENTO.ddm:L64` | `EA COD-BANCO -- COD FEBRABAN` | HIPÓTESE |
| 103 | `IBGE` | Instituto Brasileiro de Geografia e Estatística | `BENEFICIARIO.ddm:L44` | `BI COD-IBGE -- COD MUNICIPIO IBGE` | HIPÓTESE |
| 104 | `PBF` · `BPC` · `PETI` | Siglas de programas sociais citadas como exemplo | `PROGRAMA-SOCIAL.ddm:L24` | `AC SIGLA-PROGRAMA -- SIGLA (EX: PBF, BPC, PETI)` | CONFIRMADO |
| 105 | `IN-TCU 63/2010` | Instrução Normativa do TCU que obriga a trilha de auditoria | `AUDITORIA.ddm:L11` | "OBRIGATORIEDADE LEGAL: IN-TCU 63/2010". Base normativa de M-47 | CONFIRMADO |
| 106 | `Lei 8159, art. 14` | Norma que define a retenção mínima de 10 anos | `AUDITORIA.ddm:L13` | ">>> RETENCAO MINIMA: 10 ANOS (ART 14 LEI 8159) <<<" | CONFIRMADO |
| 107 | `Portaria 847/2003` | Norma que restringe alteração da ordem dos campos do DDM | `BENEFICIARIO.ddm:L11-12` | "NAO ALTERAR ORDEM DOS CAMPOS SEM AUTORIZACAO DO COMITE TECNICO CGTI/MDAS" | CONFIRMADO |
| 108 | `Portaria 213/2010` | Decisão que suspendeu a gravação de eventos de consulta | `AUDITORIA.ddm` nota final | "ACOES 'CO' NAO SAO GRAVADAS DESDE 2010 POR QUESTAO DE VOLUME" | CONFIRMADO |
| 109 | `FR-SIFAP-012` | Formulário de autorização para exceder o limite de dependentes | `REGRAS-NEGOCIO-2012.md` RN-004 | Processo manual fora do sistema | CONFIRMADO |
| 110 | `ITSM-SIFAP-002` | Norma de reorganização trimestral do arquivo de pagamentos | `PAGAMENTO.ddm` nota final | "REORGANIZACAO TRIMESTRAL OBRIGATORIA" | CONFIRMADO |
| 111 | `SYSAOS` | Utilitário Adabas Online System | `AUDITORIA.ddm` NOTA2 | Único caminho para visualizar exclusões ocultadas pelo relatório (M-47) | HIPÓTESE |
| 112 | `JES2` / `JCL` | Subsistema e linguagem de controle de job em mainframe | `AUDITORIA.ddm:L67` | `FC NOM-JOB-BATCH -- NOME JOB JES2/JCL` | HIPÓTESE |
| 113 | `ABEND U4038` | Encerramento anormal por excesso de erros no batch | `REGRAS-NEGOCIO-2012.md` §5.2 | Comportamento documentado e **não implementado** em `BATCHPGT` | CONFIRMADO |

> ⚠️ **Termos com significado em disputa.** Os itens 63-71 incluem conjuntos de códigos que
> **divergem entre DDM, código e documentação**. Não use nenhum deles como linguagem ubíqua do
> Estágio 2 antes da validação humana registrada em [`mysteries-found.md`](mysteries-found.md).

> 💡 Organize por domínio se ajudar. Adicione linhas à vontade — a meta é 30+.

---

✅ **Critério de pronto:** 30+ termos, cada um com programa-fonte, status CONFIRMADO/HIPÓTESE atribuído, hipóteses marcadas para validação com facilitador.

---

### Continuar a leitura

<table width="100%">
<tr>
<td width="50%" valign="top" align="left">
<sub><strong>← ANTERIOR</strong></sub><br/>
<a href="GUIDE.md"><strong>GUIDE do Estágio 1</strong></a><br/>
<sub>Passo a passo.</sub>
</td>
<td width="50%" valign="top" align="right">
<sub><strong>PRÓXIMO →</strong></sub><br/>
<a href="discovery-report.md"><strong>Relatório de Descoberta</strong></a><br/>
<sub>Consolidação final.</sub>
</td>
</tr>
</table>

<sub>↑ <a href="../README.md">Voltar ao Kit PT-BR</a></sub>

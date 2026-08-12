<!-- markdownlint-disable MD012 MD013 MD022 MD025 MD026 MD028 MD029 MD031 MD033 MD034 MD038 MD040 MD051 MD060 -->

# Relatório de Descoberta — Estágio 1: Arqueologia Digital

![ESTÁGIO 01 Arqueologia](https://img.shields.io/badge/ESTÁGIO-01%20Arqueologia-F25022?style=for-the-badge) ![TIPO Worksheet](https://img.shields.io/badge/TIPO-Worksheet-1A1A1A?style=for-the-badge) ![PREENCHA Durante S1](https://img.shields.io/badge/PREENCHA-Durante%20S1-737373?style=for-the-badge)

> 🗺 **Você está aqui:** [Kit PT-BR](../README.md) → [Estágio 1](README.md) → **discovery-report**

> **Para quem é isto?** Este é um **artefato preenchido pelo time** durante o Estágio 1 (Arqueologia).
>
> **O que você terá ao final do estágio:**
>
> 1. Este documento totalmente preenchido com os dados reais do legado SIFAP
> 2. Rastreabilidade para `01-arqueologia/legado-sifap/` (programas `.NSN` e DDMs)
> 3. Base de evidência usada nas EARS do Estágio 2 (`source_legacy:`)
>
> 📘 **Guia passo a passo:** [`GUIDE.md`](GUIDE.md).


> Este documento consolida todas as descobertas do Estágio 1.
> Preencha cada seção com as conclusões do time (use `/discovery-report` ao final do estágio).
> **Este é o input principal do Estágio 2** — sem ele, a especificação vira chute.

**Time**: <!-- preencher -->
**Data**: 2026-08-12
**Edição**: <!-- preencher -->
**Participantes**: <!-- preencher: 5 pares cobrindo 10 personas -->

---

## 1. Sumário Executivo

A codebase legada tem **15 programas Natural e 4 DDMs Adabas** em 30 arquivos, e todos foram lidos integralmente ([`inventory.md`](inventory.md)). A leitura produziu **156 regras candidatas**, das quais apenas **23 são confirmadas** por documentação, 76 são inferidas somente do código e 57 permanecem sem explicação ([`business-rules-catalog.md`](business-rules-catalog.md)). O sistema é **totalmente desconectado no nível de programa**: zero chamadas entre programas contra 47 acessos a dados, com dois programas que nenhum caminho de execução alcança ([`dependency-map.md`](dependency-map.md)). O maior risco registrado é **M-47** — o relatório oficial de auditoria filtra os eventos de exclusão, num arquivo cujo próprio DDM declara obrigatoriedade legal de trilha ([`mysteries-found.md`](mysteries-found.md)). A confiança é **alta quanto aos fatos do código**, pois cada regra cita `arquivo:linha` verificável, e **baixa quanto à semântica de negócio**, pois as 56 perguntas registradas seguem abertas e nenhuma recebeu validação humana.

## 2. O Que Sabemos (Confirmado)

### 2.1 Regras de Negócio

As 23 regras abaixo são as **únicas** que cruzam com a documentação histórica. Fonte: [`business-rules-catalog.md`](business-rules-catalog.md).

| # | Regra | EARS | Fonte (`arquivo:linha`) |
| -: | --- | --- | --- |
| 5 | Situação `'S'` recusa elegibilidade | Unwanted | `VALELEG.NSN:L116-121` |
| 6 | Situação `'C'` ou `'D'` recusa elegibilidade | Unwanted | `VALELEG.NSN:L122-126` |
| 7 | Situação `'I'` recusa elegibilidade | Unwanted | `VALELEG.NSN:L127-131` |
| 14 | Programa tipo trabalho exige idade ≥ 16 *(limite superior não documentado)* | Unwanted | `VALELEG.NSN:L190-196` |
| 22 | CPF é obrigatório no cadastro | Unwanted | `CADBENEF.NSN:L105-109` |
| 23 | CPF validado por dígito verificador módulo 11 | Unwanted | `CADBENEF.NSN:L112-117, L224-269` |
| 25 | Data de nascimento é obrigatória | Unwanted | `CADBENEF.NSN:L125-129` |
| 27 | CPF já cadastrado impede inclusão *(doc restringe a ativos)* | Unwanted | `CADBENEF.NSN:L143-147` |
| 29 | Inclusão atribui situação `'A'` | Event-driven | `CADBENEF.NSN:L162-164` |
| 48 | Dígito verificador inválido reprova o CPF | Unwanted | `VALBENEF.NSN:L114-120` |
| 53 | Domínio de situação restrito a `A/S/C/I/D` | Unwanted | `VALBENEF.NSN:L165-171` |
| 55 | CPF zero ou com dígito inválido é recusado | Unwanted | `VALDOCS.NSN:L68-73, L98-146` |
| 62 | Batch processa apenas beneficiários ativos | Unwanted | `BATCHPGT.NSN:L194-198` |
| 64 | Erro individual não interrompe o processamento | Unwanted | `BATCHPGT.NSN:L212-226` |
| 68 | Fator aplicado pela primeira faixa cujo teto ≥ renda *(doc diz per capita)* | Ubiquitous | `BATCHPGT.NSN:L152-162, L261-262` |
| 71 | Valores truncados em centavos, sem arredondamento | Ubiquitous | `BATCHPGT.NSN:L283-285` |
| 97 | Situação traduzida pelo domínio `A/S/C/I/D` | Ubiquitous | `CONSBENF.NSN:L109-123` |
| 101 | Cálculo exige beneficiário ativo | Unwanted | `CALCBENF.NSN:L163-166` |
| 104 | Fator aplicado pela primeira faixa cujo teto ≥ renda | Ubiquitous | `CALCBENF.NSN:L122-132, L201-202` |
| 107 | Valores truncados em centavos | Ubiquitous | `CALCBENF.NSN:L229-231` |
| 115 | Total de descontos limitado a 30% do bruto | Ubiquitous | `CALCDSCT.NSN:L102-106` |
| 118 | Desconto judicial não respeita o teto de 30% | Optional | `CALCDSCT.NSN:L130-131, L179-184` |
| 130 | Valor corrigido truncado em centavos | Ubiquitous | `CALCCORR.NSN:L153-156` |

### 2.2 Dependências

Fonte: [`dependency-map.md`](dependency-map.md) e [`dependency-map.mmd`](dependency-map.mmd).

| Métrica | Valor |
| --- | ---: |
| Arestas programa → programa | **0** |
| Arestas programa → dado | **47** |
| Chamadas intra-programa (`PERFORM`) | 23 |
| Referências quebradas no código | 4 |

Busca por `CALLNAT`, `INCLUDE`, `FETCH`, `STACK` e `CALL` nos 15 arquivos retorna zero. **A fronteira de módulo não pode ser derivada de chamadas** — só de quem escreve em qual arquivo. `VALBENEF` e `VALDOCS` declaram VIEW, não executam acesso a dados e não são alcançados por ninguém.

### 2.3 Estruturas de Dados

Fonte: [`inventory.md`](inventory.md) e [`business-rules-catalog.md`](business-rules-catalog.md).

| DDM | FNR | Volume declarado | Escritores |
| --- | ---: | --- | --- |
| `BENEFICIARIO` | 150 | ~4,2 mi registros · 52 campos | `CADBENEF`, `CADDEPEND` |
| `PROGRAMA-SOCIAL` | 151 | ~45 registros · 42 campos | `CADPROG` |
| `PAGAMENTO` | 152 | ~180 mi registros · 50 campos | `BATCHPGT`, `CALCBENF`, `BATCHCON`, `CALCCORR`, `CALCDSCT` |
| `AUDITORIA` | 153 | ~25 mi registros · 34 campos | `BATCHCON` |

> ⚠️ **As VIEWs dos programas divergem dos DDMs** em nome, tipo e existência de campo (M-12), e os números de arquivo citados nos comentários não conferem (M-48). **Nenhum schema-alvo deve ser derivado das VIEWs.**

## 3. O Que É Arriscado

### 3.1 Perguntas em Aberto Aguardando Validação Humana

**As 56 perguntas registradas em [`mysteries-found.md`](mysteries-found.md) estão com status `aberta`. Nenhuma recebeu validação humana.** As seis de maior impacto declarado:

| Pergunta aberta | Evidência (`path:linha`) | Impacto | Hipótese (não confirmada) | Pessoa/área responsável | Status |
| --- | --- | --- | --- | --- | --- |
| M-47 · Quem autorizou a exclusão dos eventos `'EX'` do relatório de auditoria? | `legado-sifap/natural-programs/RELAUDIT.NSN:L100-108`; `legado-sifap/adabas-ddms/AUDITORIA.ddm` NOTA2 | Filtro precede os filtros do operador; solicitar `'EX'` retorna nenhum registro. O DDM declara obrigatoriedade legal (IN-TCU 63/2010). | Registrada no DDM: "CUIDADO - PROGRAMA RELAUDIT.NSN FILTRA ACOES 'EX' NA EXIBICAO". **Não confirmada.** | `<preencher>` | aberta |
| M-14 · Qual a base normativa da situação suspensa acima de 75 anos? | `legado-sifap/natural-programs/CADBENEF.NSN:L166-169` | Executado também em alterações. Encadeado com `VALELEG.NSN:L116-121` e `BATCHPGT.NSN:L195-198`, resulta em inelegibilidade e ausência de pagamento. | Nenhuma hipótese registrada. | `<preencher>` | aberta |
| M-44 · Qual o significado das situações de pagamento `'P'`, `'C'` e `'E'`? | `legado-sifap/adabas-ddms/PAGAMENTO.ddm` campo `DA`; `legado-sifap/natural-programs/BATCHREL.NSN:L78-83`; `RELPGT.NSN:L126-141` | Interpretações divergentes entre DDM e relatórios. | Nenhuma hipótese registrada. | `<preencher>` | aberta |
| M-34 · Qual a codificação de região válida? | `legado-sifap/natural-programs/CALCBENF.NSN:L87-119`; `BATCHREL.NSN:L114-131`; `legado-sifap/adabas-ddms/BENEFICIARIO.ddm` campo `BJ` | Codificações incompatíveis entre código, DDM e documentação. | Nenhuma hipótese registrada. | `<preencher>` | aberta |
| M-20 · Qual a implementação de referência do cálculo de benefício e desconto? | `legado-sifap/natural-programs/BATCHPGT.NSN:L279-312` vs `CALCBENF.NSN:L219-259` vs `CALCDSCT.NSN:L99-196` | O processamento mensal não invoca os programas de cálculo. | Registrada no código: "INICIALIZA TABELA FATORES REGIONAIS (MESMA DO CALCBENF)". **Não confirmada.** | `<preencher>` | aberta |
| M-02 · A verificação de renda usa renda familiar total ou per capita? | `legado-sifap/natural-programs/VALELEG.NSN:L157-163`; `legado-sifap/adabas-ddms/PROGRAMA-SOCIAL.ddm` campo `CA RENDA-MAX-PERCAP` | Define quais beneficiários são aprovados. | Nenhuma hipótese registrada. | `<preencher>` | aberta |

As outras 50 (`M-01` … `M-56`) estão no registro completo, com os mesmos seis campos e o mesmo status.

> **HARD GATE.** Nenhuma pergunta em aberto pode fundamentar EARS, requisito ou decisão de escopo antes de validação humana explícita com evidência `path:linha`.

### 3.2 Regras com Evidência Fraca

**76 regras classificadas como Inferida** — derivadas apenas do código, sem confirmação documental. Podem fundamentar EARS **desde que o requisito registre que a fonte é o comportamento observado, não uma regra validada**. Concentrações de maior risco:

- **Fatores de cálculo** (regras 67, 69, 103, 105, 114): escadas de fator familiar, etário e alíquotas de contribuição inteiramente fixas em código.
- **Sentinelas implícitas** (regras 8, 9, 10): o valor `0` em idade mínima, idade máxima e renda máxima funciona como "sem restrição", sem contrato documentado.
- **Bordas não tratadas**: renda acima de 9.999,99 deixa o fator de renda sem atribuição (regras 68 e 104); bruto acima de 9.999,99 não recebe contribuição social (regra 114).
- **Cálculo de idade** (regra 18): obtida por subtração de anos, ignorando mês e dia — afeta os cortes de 16, 60 e 65 anos.

## 4. Hipóteses de Carving Recomendadas

> **São hipóteses, não decisões.** Derivadas da propriedade de escrita sobre cada arquivo — único critério de fronteira disponível, dada a ausência de chamadas. A avaliação e a escolha cabem ao `@architect` e ao Product Owner no Estágio 2.

### Hipótese 1: Gestão de Programas Sociais

- Programas: `CADPROG` · DDMs: `PROGRAMA-SOCIAL` (151)
- Racional: fronteira mais limpa do sistema — escritor único, 6 acessos, 45 registros a migrar.

### Hipótese 2: Elegibilidade

- Programas: `VALELEG` · DDMs: lê 150 e 151, **não escreve em nenhum**
- Racional: serviço de decisão puro, sem risco de migração de dados, mas concentra M-01 e M-02.

### Hipótese 3: Cadastro de Beneficiários e Dependentes

- Programas: `CADBENEF`, `CADDEPEND`, `VALBENEF`, `VALDOCS` · DDMs: `BENEFICIARIO` (150)
- Racional: os dois únicos escritores do arquivo ficam dentro do recorte, junto das duas rotinas órfãs.

### Hipótese 4: Processamento de Pagamento

- Programas: `BATCHPGT`, `CALCBENF`, `CALCDSCT`, `CALCCORR`, `BATCHCON` · DDMs: `PAGAMENTO` (152)
- Racional: maior valor de negócio e maior risco — cinco escritores no mesmo arquivo e 180 milhões de registros.

### Hipótese 5: Consulta e Prestação de Contas

- Programas: `CONSBENF`, `RELPGT`, `RELAUDIT`, `BATCHREL` · DDMs: somente leitura sobre 150, 152 e 153
- Racional: zero escrita, portanto zero risco de corromper dados, mas concentra os achados de exposição e auditoria.

## 5. Artefatos Fonte

| Artefato          | Caminho                                                  | Status               |
| ----------------- | -------------------------------------------------------- | -------------------- |
| Inventário        | [inventory.md](inventory.md)                             | ✅ 30 arquivos · 15/15 programas |
| Regras de Negócio | [business-rules-catalog.md](business-rules-catalog.md)   | ✅ 156 regras · 23 confirmadas · 76 inferidas · 57 mistérios |
| Dependências      | [dependency-map.md](dependency-map.md)                   | ✅ 47 arestas · 0 programa→programa |
| Perguntas em aberto | [mysteries-found.md](mysteries-found.md)               | ✅ 56 registros · 0 encerradas |
| Glossário         | [glossary.md](glossary.md)                               | ✅ 113 termos · 85 confirmados · 28 hipóteses |

**Fontes não esgotadas:** `MANUAL-TECNICO-SIFAP-2008.md` e `ARQUITETURA-ORIGINAL-1997.md` não foram cruzados com o código.

## 6. Aprovação da Equipe

- Revisado por: <!-- preencher -->
- Data: <!-- preencher: YYYY-MM-DD -->
- Confiança: <!-- preencher: Alta / Média / Baixa -->

---

✅ **Critério de pronto:** menos de 3 páginas, sumário com ≤5 frases, 3–5 hipóteses de carving, todos os artefatos fonte com status.

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
<a href="../02-spec-moderna/README.md"><strong>Estágio 2 — Spec Moderna</strong></a><br/>
<sub>Passagem H1.</sub>
</td>
</tr>
</table>

<sub>↑ <a href="../README.md">Voltar ao Kit PT-BR</a></sub>

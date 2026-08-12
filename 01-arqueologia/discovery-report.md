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
**Data**: <!-- preencher: YYYY-MM-DD -->
**Edição**: <!-- preencher -->
**Participantes**: <!-- preencher: 5 pares cobrindo 10 personas -->

---

## 1. Sumário Executivo

<!-- preencher: máximo de 5 frases — o que é o SIFAP, o que ele faz, qual o estado do código -->

## 2. O Que Sabemos (Confirmado)

### 2.1 Regras de Negócio

<!-- preencher: regras confirmadas com candidatas EARS — referencie business-rules-catalog.md -->

### 2.2 Dependências

<!-- preencher: contagens de arestas do mapa de dependências — referencie dependency-map.md -->

### 2.3 Estruturas de Dados

<!-- preencher: resumo dos DDMs Adabas analisados pelo time -->

## 3. O Que É Arriscado

### 3.1 Perguntas em Aberto Aguardando Validação Humana

Registro completo com 56 entradas em [`mysteries-found.md`](mysteries-found.md). As seis que **bloqueiam decisão de escopo**:

| Pergunta aberta | Evidência (`path:linha`) | Impacto | Hipótese (não confirmada) | Pessoa/área responsável | Status |
| --- | --- | --- | --- | --- | --- |
| M-47 · Quem autorizou a exclusão dos eventos de ação `'EX'` do relatório de trilha de auditoria? | `legado-sifap/natural-programs/RELAUDIT.NSN:L100-108`; `legado-sifap/adabas-ddms/AUDITORIA.ddm` NOTA2 e cabeçalho (`IN-TCU 63/2010`) | O filtro precede os filtros do operador; solicitar a ação `'EX'` retorna nenhum registro. O DDM declara obrigatoriedade legal da trilha. | Registrada no DDM: "CUIDADO - PROGRAMA RELAUDIT.NSN FILTRA ACOES 'EX' NA EXIBICAO". **Não confirmada.** | `<preencher>` | aberta |
| M-14 · Qual é a base normativa da suspensão automática de beneficiários com mais de 75 anos? | `legado-sifap/natural-programs/CADBENEF.NSN:L166-169` | Executado também em alterações. Encadeado com `VALELEG.NSN:L116-121` e `BATCHPGT.NSN:L195-198`, resulta em inelegibilidade e ausência de pagamento. | Nenhuma hipótese registrada. | `<preencher>` | aberta |
| M-44 · Qual é o significado das situações de pagamento `'P'`, `'C'` e `'E'`? | `legado-sifap/adabas-ddms/PAGAMENTO.ddm` campo `DA`; `legado-sifap/natural-programs/BATCHREL.NSN:L78-83`; `RELPGT.NSN:L126-141` | Interpretações divergentes entre DDM e relatórios. | Nenhuma hipótese registrada. | `<preencher>` | aberta |
| M-34 · Qual é a codificação de região válida no sistema? | `legado-sifap/natural-programs/CALCBENF.NSN:L87-119`; `BATCHREL.NSN:L114-131`; `legado-sifap/adabas-ddms/BENEFICIARIO.ddm` campo `BJ` | Codificações incompatíveis entre si em código, DDM e documentação. | Nenhuma hipótese registrada. | `<preencher>` | aberta |
| M-20 · Qual é a implementação de referência do cálculo de benefício e de desconto? | `legado-sifap/natural-programs/BATCHPGT.NSN:L279-312` vs `CALCBENF.NSN:L219-259` vs `CALCDSCT.NSN:L99-196` | O processamento mensal não invoca os programas de cálculo. | Registrada no comentário do código: "INICIALIZA TABELA FATORES REGIONAIS (MESMA DO CALCBENF)". **Não confirmada.** | `<preencher>` | aberta |
| M-02 · A verificação de renda deve usar renda familiar total ou renda per capita? | `legado-sifap/natural-programs/VALELEG.NSN:L157-163`; `legado-sifap/adabas-ddms/PROGRAMA-SOCIAL.ddm` campo `CA RENDA-MAX-PERCAP` | Define quais beneficiários são aprovados. | Nenhuma hipótese registrada. | `<preencher>` | aberta |

> **HARD GATE.** Nenhuma dessas perguntas pode fundamentar EARS, requisito ou decisão de escopo
> antes de validação humana explícita com evidência `path:linha`.

### 3.2 Regras com Evidência Fraca

**76 regras classificadas como Inferida** — derivadas apenas do código, sem confirmação documental. Elas podem fundamentar EARS **desde que o requisito registre explicitamente que a fonte é o comportamento observado, não uma regra de negócio validada**.

Concentrações de maior risco:

- **Fatores de cálculo** (regras 67, 69, 103, 105, 114): escadas de fator familiar, etário e alíquotas de contribuição estão inteiramente fixas em código, sem nenhuma referência documental.
- **Sentinelas implícitas** (regras 8, 9, 10): o valor `0` em idade mínima, idade máxima e renda máxima funciona como "sem restrição". Não há documentação desse contrato.
- **Comportamento de borda não tratado**: renda acima de 9.999,99 deixa o fator de renda sem atribuição (regras 68 e 104); bruto acima de 9.999,99 não recebe contribuição social (regra 114).
- **Defeito de cálculo de idade** (regra 18): a idade é obtida por subtração de anos, ignorando mês e dia. Afeta diretamente os cortes de 16, 60 e 65 anos.

## 4. Hipóteses de Carving Recomendadas

Derivadas da propriedade de escrita sobre cada arquivo — o único critério de fronteira disponível, dada a ausência de chamadas entre programas.

### Hipótese 1: Gestão de Programas Sociais

- Programas: `CADPROG`
- DDMs: `PROGRAMA-SOCIAL` (151)
- Racional: **Fronteira mais limpa do sistema.** Único escritor, 6 acessos totais, 5 regras catalogadas. Nenhum outro programa grava neste arquivo. Carrega apenas um mistério bloqueante (M-17, o `FATOR-K`). Menor risco de migração de dados: 45 registros.

### Hipótese 2: Elegibilidade

- Programas: `VALELEG`
- DDMs: lê `BENEFICIARIO` (150) e `PROGRAMA-SOCIAL` (151); **não escreve em nenhum**
- Racional: Serviço de decisão puro, sem efeito colateral em dados — elimina o risco de migração. 20 regras catalogadas, com 4 confirmadas e rascunho EARS pronto. Contrapartida: concentra M-01 e M-02, ambos bloqueantes, e a lógica **não é consumida pelo batch de pagamento**, o que reduz seu valor demonstrável isoladamente.

### Hipótese 3: Cadastro de Beneficiários e Dependentes

- Programas: `CADBENEF`, `CADDEPEND`, `VALBENEF`, `VALDOCS`
- DDMs: `BENEFICIARIO` (150)
- Racional: Agrupamento coeso — os dois únicos escritores do arquivo estão dentro do recorte, junto com as duas rotinas de validação órfãs. Alto valor de negócio. Contrapartida: carrega M-14, M-15, M-16, M-18 e M-19, incluindo dois backdoors de validação de CPF, e o maior volume de dados a migrar.

### Hipótese 4: Processamento de Pagamento

- Programas: `BATCHPGT`, `CALCBENF`, `CALCDSCT`, `CALCCORR`, `BATCHCON`
- DDMs: `PAGAMENTO` (152), lê 150 e 151, escreve 153
- Racional: **Maior valor de negócio e maior risco.** Cinco escritores no mesmo arquivo, dois criando registros sem coordenação, lógica de cálculo duplicada e 180 milhões de registros sem política de expurgo. Concentra M-20, M-24, M-27, M-29, M-36, M-38, M-39 e M-43. Não recomendado como primeiro recorte.

### Hipótese 5: Consulta e Relatório

- Programas: `CONSBENF`, `RELPGT`, `RELAUDIT`, `BATCHREL`
- DDMs: somente leitura sobre 150, 152 e 153
- Racional: Zero escrita, portanto zero risco de corromper dados. Expõe problemas de exposição de dado pessoal (M-32, M-46) e o achado de auditoria (M-47). Bom candidato se o objetivo do recorte for demonstrar conformidade, não regra de negócio.

> **A escolha do recorte é decisão do Product Owner**, registrada em
> [`02-spec-moderna/scope-decisions.md`](../02-spec-moderna/scope-decisions.md). Este relatório
> apresenta as opções e seus custos; não seleciona nenhuma.

## 5. Artefatos Fonte

| Artefato          | Caminho                                                  | Status               |
| ----------------- | -------------------------------------------------------- | -------------------- |
| Inventário        | [inventory.md](inventory.md)                             | ✅ 30 arquivos · 4 diretórios · 100% |
| Regras de Negócio | [business-rules-catalog.md](business-rules-catalog.md)   | ✅ 156 regras · 15/15 programas · 4/4 DDMs |
| Dependências      | [dependency-map.md](dependency-map.md)                   | ✅ 47 arestas · 0 programa→programa · 4 referências quebradas |
| Perguntas em aberto | [mysteries-found.md](mysteries-found.md)               | ✅ 56 registros · 0 encerradas |
| Glossário         | [glossary.md](glossary.md)                               | ✅ 113 termos · 85 confirmados · 28 hipóteses |

**Fontes não esgotadas:** `MANUAL-TECNICO-SIFAP-2008.md` e `ARQUITETURA-ORIGINAL-1997.md` ainda não foram cruzados com o código. Podem confirmar ou contradizer regras hoje classificadas como Inferida.

## 6. Aprovação da Equipe

- Revisado por: `<preencher>`
- Confiança: **Alta quanto aos fatos do código · Baixa quanto à semântica de negócio**

Justificativa da confiança dividida: a cobertura de leitura é integral e cada uma das 156 regras cita `arquivo:linha` verificável por terceiros. Porém, 57 regras têm intenção desconhecida, 56 perguntas seguem abertas e a única documentação de negócio disponível é um rascunho de 2012, declarado incompleto por sua própria autora e contraditório com o código em pontos verificados. **O time sabe o que o sistema faz; não sabe por quê.**

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

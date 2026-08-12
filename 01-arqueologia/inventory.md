<!-- markdownlint-disable MD012 MD013 MD022 MD025 MD026 MD028 MD029 MD031 MD033 MD034 MD038 MD040 MD051 MD060 -->

# Inventário Legado — Time `<preencher>`

![ESTÁGIO 01 Arqueologia](https://img.shields.io/badge/ESTÁGIO-01%20Arqueologia-F25022?style=for-the-badge) ![TIPO Worksheet](https://img.shields.io/badge/TIPO-Worksheet-1A1A1A?style=for-the-badge) ![PREENCHA Durante S1](https://img.shields.io/badge/PREENCHA-Durante%20S1-737373?style=for-the-badge)

> 🗺 **Você está aqui:** [Kit PT-BR](../README.md) → [Estágio 1](README.md) → **inventory**

> **Estágio 1 · Passo 1 (`/archaeology-kickoff`)**
>
> ⚠️ **Primeira passada.** Este inventário é feito SEM abrir nenhum programa.
> Trabalhe apenas sobre nomes de arquivo e estrutura de pastas. Ele será revisado
> conforme o time rodar `/extract-business-rules`, `/map-dependencies` e
> `/catalog-mysteries` durante o Estágio 1.
>
> 📘 **Guia passo a passo:** [`GUIDE.md`](GUIDE.md).

**Data:** 2026-08-12
**Autor da varredura:** `@archaeologist-agent` (orientação) + par responsável: `<preencher>`
**Caminho escaneado:** `01-arqueologia/legado-sifap/`

---

## Estrutura de Pastas

**Total: 4 diretórios · 30 arquivos.**

```text
01-arqueologia/legado-sifap/
├── COMO-LER-NATURAL.md
├── README.md
├── adabas-ddms/            (5 arquivos)
│   ├── AUDITORIA.ddm
│   ├── BENEFICIARIO.ddm
│   ├── PAGAMENTO.ddm
│   ├── PROGRAMA-SOCIAL.ddm
│   └── README.md
├── legacy-docs/            (7 arquivos)
│   ├── ARQUITETURA-ORIGINAL-1997.docx
│   ├── ARQUITETURA-ORIGINAL-1997.md
│   ├── MANUAL-TECNICO-SIFAP-2008.docx
│   ├── MANUAL-TECNICO-SIFAP-2008.md
│   ├── REGRAS-NEGOCIO-2012.docx
│   ├── REGRAS-NEGOCIO-2012.md
│   └── README.md
└── natural-programs/       (16 arquivos)
    ├── BATCHCON.NSN
    ├── BATCHPGT.NSN
    ├── BATCHREL.NSN
    ├── CADBENEF.NSN
    ├── CADDEPEND.NSN
    ├── CADPROG.NSN
    ├── CALCBENF.NSN
    ├── CALCCORR.NSN
    ├── CALCDSCT.NSN
    ├── CONSBENF.NSN
    ├── RELAUDIT.NSN
    ├── RELPGT.NSN
    ├── VALBENEF.NSN
    ├── VALDOCS.NSN
    ├── VALELEG.NSN
    └── README.md
```

> Verificação independente (2ª pessoa do time):
> `find 01-arqueologia/legado-sifap -type f | wc -l` → deve retornar `30`.

## Contagem de Arquivos por Tipo

| Extensão | Contagem | Propósito Provável                                                      |
| -------- | -------- | ----------------------------------------------------------------------- |
| `.NSN`   | 15       | Programas fonte Natural (unidades de compilação: program ou subprogram) |
| `.ddm`   | 4        | Data Definition Modules — visão Natural sobre um arquivo Adabas         |
| `.md`    | 8        | Documentação — 4 `README.md` de navegação + 3 docs históricos + 1 guia  |
| `.docx`  | 3        | Documentação histórica em binário (par `.docx`/`.md` do mesmo título)   |
| **Total**| **30**   |                                                                         |

> ⚠️ Ausências notáveis: **nenhum `.cpy` (copycode) e nenhum `.map` (tela de terminal)**.
> Em codebases Natural isso é raro. Ou o kit os omitiu, ou as definições de dados
> compartilhadas estão inline nos `.NSN`. **Confirmar ao abrir o primeiro programa.**

## Padrões de Convenção de Nomes

Os nomes não usam delimitador (`-` ou `_`): seguem o formato clássico Natural de
**≤ 8 caracteres**, no padrão `VERBO + OBJETO`. Agrupando pelo prefixo de verbo:

| Prefixo | Contagem | Arquivos                              | Hipótese (⚠️ não confirmada)                                                                     |
| ------- | -------- | ------------------------------------- | ------------------------------------------------------------------------------------------------ |
| `BATCH` | 3        | BATCHCON, BATCHPGT, BATCHREL          | Entry points batch. Em Natural, jobs batch normalmente são `program` (não `subprogram`) e leem entrada sequencial com `AT END OF DATA`. **Candidatos a raiz do call graph.** |
| `CAD`   | 3        | CADBENEF, CADDEPEND, CADPROG          | Abreviação PT-BR de *cadastro* → manutenção de dados (`STORE` / `UPDATE` / `DELETE`).             |
| `CALC`  | 3        | CALCBENF, CALCCORR, CALCDSCT          | Cálculo. Provável uso de campos `P` (packed decimal) — típico em lógica financeira.               |
| `VAL`   | 3        | VALBENEF, VALDOCS, VALELEG            | Validação. Candidatos a `subprogram` chamados via `CALLNAT` por vários outros programas.          |
| `REL`   | 2        | RELAUDIT, RELPGT                      | Abreviação PT-BR de *relatório* → provável saída com `WRITE` / `DISPLAY` e control-break (`AT BREAK`). |
| `CONS`  | 1        | CONSBENF                              | Abreviação PT-BR de *consulta* → provável programa online/interativo. **Único do grupo** — ver Itens Incomuns. |

### Padrão secundário — sufixo de objeto (mais revelador que o prefixo)

O sufixo indica **sobre qual entidade** o verbo opera, e cruza direto com os `.ddm`:

| Sufixo             | Aparece em                                        | Cruza com o DDM        |
| ------------------ | ------------------------------------------------- | ---------------------- |
| `BENEF` / `BENF`   | CADBENEF, VALBENEF, CONSBENF, CALCBENF (**4×**)   | `BENEFICIARIO.ddm`     |
| `PGT`              | BATCHPGT, RELPGT (2×)                             | `PAGAMENTO.ddm`        |
| `PROG`             | CADPROG (1×)                                      | `PROGRAMA-SOCIAL.ddm`  |
| `AUDIT`            | RELAUDIT (1×)                                     | `AUDITORIA.ddm`        |

> 📌 **Leitura do padrão:** os 4 DDMs têm cobertura de programa. `BENEFICIARIO` é a
> entidade mais tocada (4 programas) → forte candidata a núcleo do domínio.
> `AUDITORIA` só aparece num relatório → possível gravação implícita dentro de
> outros programas. **Verificar no `/map-dependencies`.**

## Itens Incomuns (Top 3)

| #   | Caminho do Arquivo                                          | O Que o Torna Incomum                                                                                                                                    | Investigação Sugerida                                                                                                                 |
| --- | ----------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | `01-arqueologia/legado-sifap/natural-programs/CADDEPEND.NSN` | **Único nome com 9 caracteres.** Os outros 14 têm ≤ 8, o limite clássico de nome de objeto Natural. Também é o único cujo objeto (`DEPEND`) **não tem DDM correspondente**. | Abrir e verificar se `DEPEND` (dependente?) é armazenado dentro de `BENEFICIARIO.ddm` como grupo `PE`/`MU`. Se sim, é modelagem em array — impacto direto no schema-alvo. |
| 2   | `01-arqueologia/legado-sifap/natural-programs/CALCBENF.NSN`  | **Inconsistência de abreviação:** `BENF` (4 letras) contra `BENEF` nos outros três (CADBENEF, VALBENEF, CONSBENF).                                        | Confirmar se opera sobre a mesma entidade ou outra. Divergência de nome costuma marcar código escrito em época/equipe diferente — pista de regra de negócio divergente. |
| 3   | `01-arqueologia/legado-sifap/natural-programs/CONSBENF.NSN`  | **Prefixo `CONS` ocorre uma única vez.** Todos os outros verbos têm 2-3 programas. É o provável único caminho online/interativo do sistema.               | Verificar se tem `INPUT`/`MAP` (tela de terminal). Sendo o único ponto de entrada de usuário, é candidato natural ao recorte fino do Estágio 2. |

### Menções honrosas (não entram no Top 3)

- `legacy-docs/` — cada doc existe em `.docx` **e** `.md`. Confirmar se são
  equivalentes ou se o `.md` é conversão parcial. Datas (**1997 / 2008 / 2012**)
  mostram 3 gerações de documentação e ~14 anos de silêncio até hoje: trate como
  **possivelmente desatualizada**. O código é a verdade; o doc é hipótese.
- `PROGRAMA-SOCIAL.ddm` — único DDM com hífen no nome, escapando do padrão de
  palavra única dos outros três.

## Ordem de Leitura Proposta

> ⚠️ **Isto é hipótese, não roteiro fixo.** A ordem real vai mudar assim que o time
> rodar `/map-dependencies` e enxergar as arestas reais de `CALLNAT`.

| Ordem | O quê                                                            | Por quê                                                                                                        |
| ----- | ---------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| 1️⃣    | `BENEFICIARIO.ddm` → `PAGAMENTO.ddm` → `PROGRAMA-SOCIAL.ddm` → `AUDITORIA.ddm` | **Dados antes de código.** Sem saber os campos e descritores (`DE`, `MU`, `PE`, `SU`), a lógica dos programas fica ilegível. Comece por `BENEFICIARIO` — é a entidade mais referenciada. |
| 2️⃣    | `BATCHCON.NSN`, `BATCHPGT.NSN`, `BATCHREL.NSN`                    | **Entry points.** Batch jobs costumam ser raiz do call graph: ler daqui revela a cadeia `CALLNAT` para baixo.   |
| 3️⃣    | `VALBENEF.NSN`, `VALDOCS.NSN`, `VALELEG.NSN`                      | **Prováveis folhas mais reutilizadas.** Validações tendem a ser `subprogram` chamado por muitos — alta densidade de regra de negócio por linha. |
| 4️⃣    | `CALCBENF.NSN`, `CALCCORR.NSN`, `CALCDSCT.NSN`                    | **Coração financeiro.** Onde moram as fórmulas e o packed decimal. Fonte principal de regras para o catálogo.   |
| 5️⃣    | `CADBENEF.NSN`, `CADDEPEND.NSN`, `CADPROG.NSN`, `CONSBENF.NSN`, `RELAUDIT.NSN`, `RELPGT.NSN` | Manutenção, consulta e relatórios — dependem do entendimento construído nos passos anteriores.               |
| 📄    | `legacy-docs/*` (consulta cruzada)                                | Use **depois** de ler o código, para confrontar doc × realidade. Divergências viram registros em `mysteries-found.md`. |

### Divisão de programas por par (oficial)

> Fonte: [`GUIDE.md`](GUIDE.md) e [`LEGACY-EXPLORATION-CHECKLIST.md`](LEGACY-EXPLORATION-CHECKLIST.md).
> Esta atribuição é a do kit — **não improvise outra**. Nenhum programa pode ficar sem leitor.

| Par                   | Programas atribuídos                          | Status de leitura |
| --------------------- | --------------------------------------------- | ----------------- |
| 1 · Visão             | `CADBENEF`, `CADDEPEND`, `CADPROG`            | ⬜ ⬜ ⬜            |
| 2 · Arquitetura       | `BATCHPGT`, `BATCHREL`, `BATCHCON`            | ⬜ ⬜ ⬜            |
| 3 · Implementação     | `CALCBENF`, `CALCCORR`, `CALCDSCT`            | ⬜ ⬜ ⬜            |
| 4 · Qualidade         | `VALBENEF`, `VALDOCS`, `VALELEG`              | ⬜ ⬜ ✅            |
| 5 · Operações         | `CONSBENF`, `RELPGT`, `RELAUDIT`              | ⬜ ⬜ ⬜            |

**Cobertura atual: 1 de 15 programas (7%).** `VALELEG.NSN` está registrado em
[`business-rules-catalog.md`](business-rules-catalog.md).

---

✅ **Critério de pronto:** inventário existe, contagens precisas, 3+ padrões de nome identificados, 3 itens incomuns sinalizados.

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
<a href="business-rules-catalog.md"><strong>Catálogo de Regras</strong></a><br/>
<sub>Passo 2.</sub>
</td>
</tr>
</table>

<sub>↑ <a href="../README.md">Voltar ao Kit PT-BR</a></sub>

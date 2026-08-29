# ORCID Manual Entry — OMP plugin

[![OMP](https://img.shields.io/badge/OMP-3.5-brightgreen)](https://pkp.sfu.ca/omp/)
[![Version](https://img.shields.io/badge/version-1.0.2.1-blue)](version.xml)
[![License](https://img.shields.io/badge/license-GPL--3.0-lightgrey)](LICENSE)

**⬇️ Install package:** [OMP 3.5](https://github.com/OJSBR/orcidManualEntryOmp/releases/download/1.0.2.1/orcidManualEntry-1.0.2.1.tar.gz) — or browse all [Releases](../../releases).

A generic plugin for **Open Monograph Press (OMP)** that restores a **typeable (manual)
ORCID field** in the author/contributor form — the behaviour from older OMP versions — for
presses where **ORCID authentication (OAuth) is not configured**.

> ⚠️ **Manual entry is NOT the recommended way to collect ORCID iDs.** The recommended
> approach remains **authenticated ORCID (OAuth)**, where the author signs in at ORCID and
> the iD is verified at the source. A manually typed iD is unverified — it can be mistyped
> or belong to someone else. Use this plugin only as a fallback while your press cannot
> enable ORCID OAuth; once you configure OAuth, the plugin goes inert and OMP takes over.
> See [Why authenticated ORCID is recommended](#why-authenticated-orcid-is-recommended).

> **Developed and maintained by [OJSBR](https://ojsbr.com.br).** See the
> [Credits & authorship](#credits--authorship) section below.

## Compatibility & branches

| OMP version | Branch | Plugin release |
|-------------|--------|----------------|
| OMP 3.5.x   | [`stable-3_5_0`](../../tree/stable-3_5_0) *(default)* | 1.0.2.1 |

**Looking for the OJS edition?** It lives in its own repository,
[OJSBR/orcidManualEntry](https://github.com/OJSBR/orcidManualEntry). This repository is the same plugin with the
terminology and the data model of a book publisher.


## Why authenticated ORCID is recommended

Since PKP 3.4/3.5 the old ORCID plugin was merged into the core and the author ORCID field
became **read-only**: it can only be filled by **OAuth authentication** (the author signs in
at ORCID and authorises the press). This is deliberate — an **authenticated iD is verified
at the source**, so you know it really belongs to that person. A **manually typed iD is not
verified**: it may be mistyped, invented, or belong to someone else, and it will not carry
ORCID's "verified" status.

**So the recommendation is: enable ORCID OAuth.** See PKP's
[ORCID in OJS/OMP/OPS guide](https://docs.pkp.sfu.ca/orcid/). Use this plugin **only** while
you genuinely cannot enable OAuth (e.g. no ORCID member/public API credentials yet) and still
need to record iDs. It is a pragmatic fallback, not a replacement for authentication.

## What it does

- Acts **only** when ORCID OAuth is **not** enabled for the context. If you configure OAuth
  later, the plugin becomes inert and the core takes over the verified flow.
- Adds a plain **ORCID** text field to the contributor form (submission wizard and
  *Edit contributor*).
- Accepts the bare iD (`0000-0002-1825-0097`) or the URL, and **normalizes** it to the
  canonical `https://orcid.org/0000-0002-1825-0097` the core expects.
- Keeps the core's **format + checksum validation**: an invalid iD is rejected.
- Shows the stored iD again when *Edit contributor* is reopened, so re-saving a contributor
  never wipes it.

## Installation

1. Install via **Settings → Website → Plugins → Upload A New Plugin**, or extract the folder
   into `plugins/generic/` so you get `plugins/generic/orcidManualEntry/`.
2. Enable **ORCID manual (digitável)** under the *Generic* plugins list.

> **Keep the folder name.** PKP 3.5 derives the plugin's class namespace from its installation
> directory, so the folder must be `orcidManualEntry` — the `Omp` suffix belongs to the repository
> name only. The release tarball already unpacks with the right name.

## How it works (technical)

The core blocks manual ORCID in four places; the plugin neutralizes each **only when OAuth
is off**:

1. `Form::config::before` → adds a typeable `orcid` field to the `ContributorForm`.
2. `TemplateManager::display` → publishes `js/orcidManualEntry.js`, which registers the
   `field-orcid-manual` Vue component. This one is not obvious: `ContributorsListPanel`
   `.openEditModal()` fills every field with `field.value = contributor[name]`, **except** a
   field named `orcid`, which instead receives `field.orcid`, because the core assumes the
   `FieldOrcid` (OAuth) component there. A plain `FieldText` reads `value`, so the stored iD
   never reached the input: the form always reopened blank, and the next *Save* wrote that
   blank over the stored iD. The component extends `FieldText` and seeds `value` from the
   `orcid` prop on `mounted()`.
3. `Author::validate` → removes the `cannotUpdateAuthorOrcid` block while keeping the core's
   format/checksum validation.
4. `Author::add::before` / `Author::edit` → normalizes and (re)injects the iD before it is
   written, since the edit endpoint strips `orcid` from the parameters by default. Clearing
   an existing iD is logged to the PHP error log, so that a future regression of the Vue
   component is traceable instead of silent.

## Languages

Ships in **7 languages**: English, Portuguese (Brazil), Portuguese (Portugal), Spanish, French,
Italian and German. Note the French folder is `locale/fr` — PKP 3.5 has no `fr_FR` locale, so a
`fr_FR` folder would never be loaded.

## Tests

Verified on **OMP 3.5.0.4** against a live press: `0000-0002-1825-0097` normalises to the full
URL and validates; `0000-0002-1825-0098` (wrong checksum) and `abacaxi` are rejected. Driving
the real `Author::validate` hook, a valid ORCID clears the core's block and an invalid checksum
keeps the error.

## Credits & authorship

- **Developed and maintained by** [OJSBR](https://ojsbr.com.br) — original plugin.
- Distributed under the **GNU GPL v3**, the same license as OMP.

## Contributing

Issues and pull requests are welcome.

## License

Distributed under the **GNU GPL v3**. See [`LICENSE`](LICENSE) and `docs/COPYING`.

---

## 🇧🇷 Português

Plugin genérico para o **Open Monograph Press (OMP)** que restaura um **campo ORCID digitável
(manual)** no formulário de autor/contribuidor — como nas versões anteriores do OMP — para
editoras em que a **autenticação ORCID (OAuth) não está configurada**.

> ⚠️ **O preenchimento manual NÃO é a forma recomendada de coletar iDs ORCID.** A
> recomendação continua sendo o **ORCID autenticado (OAuth)**, em que o autor faz login no
> ORCID e o iD é verificado na fonte. Um iD digitado manualmente **não é verificado** — pode
> ser digitado errado ou pertencer a outra pessoa. Use este plugin apenas como alternativa
> temporária enquanto a editora não puder habilitar o ORCID OAuth; assim que o OAuth for
> configurado, o plugin fica inerte e o OMP assume o controle.

> **Desenvolvido e mantido pela [OJSBR](https://ojsbr.com.br).**

### Por que o ORCID autenticado é o recomendado

Desde o PKP 3.4/3.5 o antigo plugin ORCID foi incorporado ao núcleo e o campo ORCID do autor
passou a ser **somente-leitura**: só pode ser preenchido via **autenticação OAuth** (o autor
faz login no ORCID e autoriza a editora). Isso é proposital — um **iD autenticado é
verificado na origem**, então você sabe que ele realmente pertence àquela pessoa. Um **iD
digitado manualmente não é verificado**: pode conter erro de digitação, ser inventado ou ser
de outra pessoa, e não recebe o status de "verificado" do ORCID.

**Portanto, a recomendação é: habilitar o ORCID OAuth** (veja o
[guia de ORCID da PKP](https://docs.pkp.sfu.ca/orcid/)). Use este plugin **apenas** enquanto
realmente não for possível habilitar o OAuth e ainda assim for preciso registrar iDs. É uma
alternativa pragmática, não um substituto da autenticação.

### O que faz

- Age **somente** quando o ORCID OAuth **não** está habilitado no contexto; se você
  configurar o OAuth depois, o plugin fica inerte.
- Adiciona um campo de texto **ORCID** ao formulário de contribuidor (assistente de
  submissão e *Editar contribuidor*).
- Aceita o iD nu (`0000-0002-1825-0097`) ou a URL e **normaliza** para
  `https://orcid.org/0000-0002-1825-0097`.
- Mantém a **validação de formato e dígito verificador** do núcleo: iD inválido é rejeitado.
- Reexibe o iD gravado ao reabrir *Editar contribuidor*, de modo que salvar o contribuidor
  de novo nunca apaga o ORCID.

### Instalação

Instale em **Configurações → Website → Plugins → Enviar um novo plugin**, ou extraia a pasta
em `plugins/generic/` (ficando `plugins/generic/orcidManualEntry/`). Depois ative o
**ORCID manual (digitável)** na lista de plugins *Genéricos*.

> **Não renomeie a pasta.** O PKP 3.5 deriva o namespace da classe do diretório de instalação,
> então a pasta precisa se chamar `orcidManualEntry` — o sufixo `Omp` é só do repositório. O pacote
> de release já descompacta com o nome certo.

### Idiomas

Em **7 idiomas**: inglês, português (Brasil), português (Portugal), espanhol, francês, italiano
e alemão. A pasta do francês é `locale/fr` — o PKP 3.5 não tem o locale `fr_FR`, então uma pasta
`fr_FR` nunca seria carregada.

### Testes

Verificado no **OMP 3.5.0.4** em uma editora real: `0000-0002-1825-0097` normaliza para a URL
completa e valida; `0000-0002-1825-0098` (dígito verificador errado) e `abacaxi` são recusados.
Pelo hook `Author::validate` de verdade, um ORCID válido remove o bloqueio do núcleo e um
checksum inválido mantém o erro.

### Créditos e autoria

- **Desenvolvido e mantido pela** [OJSBR](https://ojsbr.com.br) — plugin autoral.
- Distribuído sob a **GNU GPL v3**, a mesma licença do OMP.

### Licença

Distribuído sob a **GNU GPL v3**. Veja [`LICENSE`](LICENSE) e `docs/COPYING`.

# EdraWiki

Hub central de **documentação técnica, guias e tutoriais** da EDRA — Equipe de
Robótica Aérea da Universidade de Brasília (UnB).

O objetivo é reunir num só lugar, de fácil acesso, tudo que hoje está espalhado
pelos repositórios e no grupo do WhatsApp — tanto para os membros atuais quanto
para o pessoal novo que entra na equipe.

> **Site:** depois do primeiro deploy, a wiki fica publicada em
> `https://edra-unb-fga.github.io/EdraWiki/`

## O que tem aqui

- **Mapa da Documentação** — índice que aponta para toda a documentação existente
  nos outros repositórios da organização (nada é duplicado; a wiki centraliza os
  ponteiros e vai absorvendo o conteúdo aos poucos).
- **Guias técnicos** — simulação, setup, visão computacional, hardware, etc.

## Como rodar localmente

```bash
git clone https://github.com/edra-unb-fga/EdraWiki.git
cd EdraWiki
pip install -r requirements.txt
mkdocs serve
```

Acesse http://127.0.0.1:8000/

## Como contribuir

Veja o guia em [`docs/contribuir.md`](docs/contribuir.md). Em resumo: crie uma
branch, adicione/edite arquivos `.md` em `docs/`, registre a página no `mkdocs.yml`
(seção `nav`) e abra um Pull Request. Ao dar merge na `main`, o site é publicado
automaticamente (GitHub Actions).

## Stack

- [MkDocs](https://www.mkdocs.org/) + [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)
- Deploy automático via GitHub Actions + GitHub Pages (mesmo padrão do
  [EdraDocs](https://github.com/edra-unb-fga/EdraDocs))

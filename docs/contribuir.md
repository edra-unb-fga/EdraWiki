# Como contribuir

A EdraWiki é feita para crescer com a equipe. Adicionar ou melhorar uma página é
simples.

## Fluxo rápido

1. **Clone o repositório:**
   ```bash
   git clone https://github.com/edra-unb-fga/EdraWiki.git
   cd EdraWiki
   ```

2. **Instale e rode localmente** (para ver suas mudanças em tempo real):
   ```bash
   pip install -r requirements.txt
   mkdocs serve
   ```
   Acesse http://127.0.0.1:8000/ — o site recarrega sozinho quando você salva.

3. **Crie uma branch:**
   ```bash
   git checkout -b docs/minha-contribuicao
   ```

4. **Adicione/edite** os arquivos `.md` dentro de `docs/`. Coloque na pasta do tema
   certo (`simulacao/`, `setup/`, `visao/`, `hardware/`…). Crie uma pasta nova se
   for um tema que ainda não existe.

5. **Registre a página** no `mkdocs.yml`, na seção `nav`, para ela aparecer no menu.

6. **Commit e push:**
   ```bash
   git add .
   git commit -m "docs: adiciona guia sobre X"
   git push origin docs/minha-contribuicao
   ```

7. **Abra um Pull Request** no GitHub. Ao dar merge na `main`, o site é **publicado
   automaticamente** (GitHub Actions → GitHub Pages).

## Boas práticas

- **Não duplique**: se a documentação já existe em outro repo, prefira **migrar**
  (trazer para cá e atualizar o [Mapa da Documentação](mapa-da-documentacao.md)) ou,
  no mínimo, linkar no mapa.
- **Escreva para quem está chegando**: assuma pouco contexto, explique os comandos.
- **Use os recursos do Material**: caixas de aviso (`!!! warning`), abas, blocos de
  código com destaque de linguagem, etc.
- **Imagens** vão em `docs/img/` (ou numa subpasta `img/` do tema).

## Recursos úteis do MkDocs Material

Blocos de aviso (admonitions):
```markdown
!!! warning "Atenção"
    Texto do aviso.

!!! tip "Dica"
    Uma dica útil.
```

Blocos de código com destaque:
````markdown
```bash
make px4_sitl gz_x500
```
````

Documentação completa do tema:
[Material for MkDocs](https://squidfunk.github.io/mkdocs-material/reference/).

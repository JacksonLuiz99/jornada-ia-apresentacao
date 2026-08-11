# jornada-ia-apresentacao

"Minha jornada com IA no desenvolvimento" — uma apresentação em scrollytelling
contando, no formato de um `git log --graph`, como venho adotando IA no
desenvolvimento: do copiar-e-colar no ChatGPT clássico até Spec-Driven
Development (SDD).

**Ao vivo:** <https://jacksonluiz99.github.io/jornada-ia-apresentacao/>

## Rodando localmente

É só HTML + CSS estático, sem build:

```sh
python3 -m http.server
```

e abrir `http://localhost:8000/index.html` (ou abrir o arquivo direto no
navegador). Carrega fontes do Google Fonts, então precisa de internet pra
renderizar com a tipografia correta.

## Estrutura

- `index.html` — markup e o JS da apresentação (tema, rolagem, rail de commits)
- `styles.css` — todo o CSS
- `CLAUDE.md` — notas de arquitetura pra quem for editar o deck

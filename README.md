# Prompts, Skills e Agentes de IA para a Promotoria de Justiça

Apostila do minicurso realizado pela ESMP em Bauru.

**Autor:** José Eduardo de Souza Pimentel — Promotor de Justiça  
**Edição:** Agosto de 2026

---

## Conteúdo

1. Markdown, XML e Placeholders para prompts de melhor qualidade
2. Engenharia de Prompt
3. Agentes do Copilot
4. Skills
5. Claude no VS Code

---

## Como executar localmente

**Pré-requisito:** Python 3.8+

```bash
# Crie e ative o ambiente virtual
python -m venv .venv
source .venv/bin/activate        # Linux/macOS
# .venv\Scripts\activate         # Windows

# Instale as dependências
pip install mkdocs-material

# Sirva localmente
mkdocs serve
```

Acesse em `http://127.0.0.1:8000`.

## Como gerar o site estático

```bash
mkdocs build
```

O site é gerado na pasta `site/` (ignorada pelo Git).

## Deploy no GitHub Pages

```bash
mkdocs gh-deploy
```

O comando faz o build e publica automaticamente na branch `gh-pages`.

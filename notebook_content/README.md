# Conference RAG Setup - Notebook Content Structure

This directory contains markdown files that will be converted into the final Jupyter notebook.

## File Convention

Each `.md` file represents a section of the notebook. Within each file:

- Regular markdown becomes **markdown cells**
- Code blocks with ` ```python ` become **code cells**
- Sections starting with `# @title` become **form code cells** (Colab-specific)

## Files (in order)

1. `00_welcome.md` - Introduction and architecture overview
2. `01_repository_setup.md` - Git clone/fork instructions
3. `02_supabase_setup.md` - Project creation and credentials
4. `03_database_schema.md` - Create tables and RLS policies
5. `04_frontend_deploy.md` - GitHub Pages deployment
6. `05_edge_functions.md` - Deploy both Edge Functions
7. `06_scrape_data.md` - Scrape conference talks
8. `07_embed_import.md` - Generate embeddings and import
9. `08_testing.md` - Test the RAG system
10. `09_reflection.md` - Learning outcomes and extensions

## Conversion Script

Run `python convert_to_notebook.py` to generate `setup.ipynb` from these markdown files.

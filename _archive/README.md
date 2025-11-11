# Archived Notebooks (v0.0.9)

This directory contains notebooks from the **v0.0.9 template-based architecture** that were replaced during the pivot to the **v0.1 discovery-based architecture**.

## Why Archived?

ctxt underwent a fundamental architecture change in v0.1:

- **v0.0.9 (archived):** Template processor with tag substitution
- **v0.1+ (current):** Context discovery engine with automatic environment detection

The old system was working and useful, but incompatible with the new vision.

## Preserved Notebooks

- `02_tags.ipynb` - Tag discovery and substitution engine
- `03_templates.ipynb` - Template processing and file generation
- `04_cli_old.ipynb` - Original CLI for template system

## Access Old Version

The complete v0.0.9 system is preserved at git tag `v0.0.9-template-mvp`:

```bash
# View old code
git checkout v0.0.9-template-mvp

# Install old version
pip install git+https://github.com/civvic/ctxt.git@v0.0.9-template-mvp
```

## What Was Kept?

- `01_config.ipynb` - Config loading logic reused for `.ctxt/` discovery (refactored)
- `index.ipynb` - Updated for new architecture

## Migration Notes

The new v0.1 architecture is **not backward compatible** with v0.0.9 config files. The `.ctxt.yml` format used by v0.0.9 will be discovered as a context source in v0.1, but not processed as templates.

**Archived:** 2025-11-11 (M0: Transition & Preservation)


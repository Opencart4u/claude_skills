# claude_skills

Προσωπικός Claude Code skills marketplace για OpenCart projects (digital4u).

## Plugin: `oc-tools`

| Skill | Πότε ενεργοποιείται |
|---|---|
| `file-ownership` | Σε κάθε file/folder operation — ελέγχει & διορθώνει root ownership. |
| `ocmod-modifications` | Όταν γράφω/διορθώνω OCMOD `install.xml` modifications για OpenCart. |

## Εγκατάσταση σε ένα μηχάνημα

Μέσα στο Claude Code (όχι στο shell):

```
/plugin marketplace add Opencart4u/claude_skills
/plugin install oc-tools@claude_skills
```

## Ενημέρωση μετά από αλλαγές

Edit στο repo → `git push`. Μετά, στα μηχανήματα:

```
/plugin marketplace update claude_skills
```

Επιβεβαίωση ότι φορτώθηκαν: `/reload-plugins` και `/plugin`.

## Δομή

```
claude_skills/
├── .claude-plugin/
│   └── marketplace.json        ← κατάλογος plugins
├── README.md
└── oc-tools/                   ← το plugin
    ├── .claude-plugin/
    │   └── plugin.json
    └── skills/
        ├── file-ownership/
        │   └── SKILL.md
        └── ocmod-modifications/
            └── SKILL.md
```

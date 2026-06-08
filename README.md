# claude_skills

Προσωπικός **Claude Code skills marketplace** για OpenCart projects (digital4u).
Ένα κεντρικό σημείο: γράφω/ενημερώνω τα skills εδώ, και κάθε μηχάνημα τα τραβάει από το repo.

> Repo: **`Opencart4u/claude_skills`** · marketplace: **`claude_skills`** · plugin: **`oc-tools`**

---

## Plugin: `oc-tools`

| Skill | Πότε ενεργοποιείται |
|---|---|
| `file-ownership` | Σε κάθε file/folder operation — ελέγχει & διορθώνει root ownership. |
| `ocmod-modifications` | Όταν γράφω/διορθώνω OCMOD `install.xml` modifications για OpenCart. |

---

## Εγκατάσταση σε ένα μηχάνημα

Μέσα στο **Claude Code prompt** (όχι στο shell):

```
/plugin marketplace add Opencart4u/claude_skills
/plugin install oc-tools@claude_skills
```

Επιβεβαίωση φόρτωσης: `/reload-plugins` (δείχνει «Reloaded X plugins, 2 skills…») ή `/plugin` → tab **Installed**.

Shell ισοδύναμα (για headless/scripts):

```bash
claude plugin marketplace add Opencart4u/claude_skills
claude plugin install oc-tools@claude_skills
```

---

## Καθημερινή χρήση

- **Προσθήκη/αλλαγή skill:** edit στο repo → `git push` → στα μηχανήματα `/plugin marketplace update claude_skills`.
- Μετά το install, φορτώνονται **αυτόματα σε κάθε νέο session** — δεν κάνεις τίποτα.

---

## SSH / πολλά μηχανήματα — τι ισχύει ανά server

**Το install γίνεται μία φορά ανά SERVER, όχι ανά project/session.** Το plugin αποθηκεύεται στο
`~/.claude` του χρήστη του server.

| Σενάριο | Χρειάζεται install; |
|---|---|
| Ίδιος server, ξανά SSH / άλλα projects / νέα sessions | ❌ Όχι — μένει στο `~/.claude`, φορτώνει αυτόματα |
| **Καινούργιος** server (πρώτη φορά) | ✅ Ναι, μία φορά τα δύο `/plugin` commands |
| Άλλαξε κάτι στο repo (`git push`) | Στους servers: `/plugin marketplace update claude_skills` |

⚠️ **Εφήμεροι servers** (containers/VMs που στήνονται καθαροί κάθε φορά, ή χάνεται το `~/.claude`):
εκεί το install χάνεται και ξαναχρειάζεται. Αυτοματοποίησέ το στο provisioning / `.bashrc` / dotfiles
(τρέχουν headless, χωρίς interactive prompt):

```bash
claude plugin marketplace add Opencart4u/claude_skills
claude plugin install oc-tools@claude_skills
```

⚠️ **Auth:** αν το repo είναι **private**, το `marketplace add` σε νέο server θέλει git auth (GitHub
login / credential manager / SSH key / token). Αν είναι **public**, δεν χρειάζεται τίποτα.

---

## Προσθήκη νέου skill

1. Φτιάξε φάκελο `oc-tools/skills/<name>/` και μέσα ΥΠΟΧΡΕΩΤΙΚΑ ένα `SKILL.md`.
2. Ο φάκελος = το `name`. Το `description` πρέπει να λέει καθαρά «πότε χρησιμοποιείται» — βάσει αυτού
   αποφασίζει το Claude αν θα το φορτώσει.
3. `git push` → στα μηχανήματα `/plugin marketplace update claude_skills`.

Μορφή `SKILL.md`:

```markdown
---
name: kebab-case-όνομα
description: Πότε/γιατί ενεργοποιείται. Σαφές — βάσει αυτού το φορτώνει το Claude.
---

# Τίτλος

Οδηγίες / βήματα / κανόνες που θέλω να ακολουθεί το Claude όταν τρέξει αυτό το skill.
Μπορεί να έχει και βοηθητικά αρχεία στον ίδιο φάκελο (scripts, templates) και να τα αναφέρει.
```

---

## Δομή repo

```
claude_skills/
├── .claude-plugin/
│   └── marketplace.json        ← κατάλογος plugins (source: ./oc-tools)
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

---

## Σημειώσεις

- Υπάρχουν **δύο διαφορετικά** συστήματα «Skills»: τα **claude.ai dashboard Skills** (μόνο web/desktop & API)
  και τα **Claude Code CLI Skills** (τοπικά `SKILL.md`). **Δεν** συγχρονίζονται μεταξύ τους — αυτό το repo
  αφορά τα **CLI** skills.
- Επίπεδα φόρτωσης CLI skills: **Plugin** (αυτή η λύση, παντού όπου εγκατασταθεί) · **User**
  (`~/.claude/skills/`) · **Project** (`<project>/.claude/skills/`).

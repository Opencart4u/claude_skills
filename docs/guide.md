# Handoff: Δημιουργία προσωπικού Claude Code Skills Marketplace (OC projects)

> Κόλλησε ΟΛΟ αυτό το αρχείο στην αρχή ενός **καινούργιου καθαρού session** του Claude Code και
> πες: «Ακολούθησε αυτές τις οδηγίες και στήσε το repo». Είναι αυτόνομο — δεν χρειάζεται προηγούμενο context.

---

## 1. Στόχος

Θέλω **ένα κεντρικό σημείο** όπου γράφω/ενημερώνω τα δικά μου Claude Code skills (κυρίως για
OpenCart projects), και **σε όποιο μηχάνημα μπαίνω** να τα έχω διαθέσιμα/φορτωμένα — με ελάχιστο setup.

Το πρώτο skill που θέλω να μπει λέγεται **`file-ownership`** (θα κάνω paste το περιεχόμενό του στο νέο session).

---

## 2. Κρίσιμη γνώση — μη χαθεί

- Υπάρχουν **δύο διαφορετικά** συστήματα «Skills»:
  1. **claude.ai dashboard Skills** → ισχύουν ΜΟΝΟ για claude.ai web/desktop & το API.
  2. **Claude Code CLI Skills** → τοπικά αρχεία `SKILL.md`. ΑΥΤΑ θέλω.
- **Δεν υπάρχει** sync από το dashboard προς το CLI. Ό,τι φτιάχνω στο dashboard ΔΕΝ φτάνει ποτέ στο CLI.
- Άρα το «κεντρικό» για το CLI = **ένα git repo που λειτουργεί ως plugin marketplace**.
  Το γράφω/ενημερώνω σε ΕΝΑ σημείο (το repo) και κάθε μηχάνημα το τραβάει από εκεί.
- Επίπεδα φόρτωσης skills στο CLI:
  | Επίπεδο | Path | Πότε φορτώνει |
  |---|---|---|
  | Plugin (marketplace) | μέσω installed plugin | παντού όπου εγκατασταθεί — **αυτή η λύση** |
  | User | `~/.claude/skills/<name>/SKILL.md` | όλα τα projects του μηχανήματος |
  | Project | `<project>/.claude/skills/<name>/SKILL.md` | μόνο σε ένα project |

---

## 3. Τι θέλω να φτιαχτεί (δομή repo)

Όνομα repo (πρότεινε/άλλαξέ το): **`my-oc-skills`** · όνομα plugin: **`oc-tools`**

```
my-oc-skills/                       ← git repo (θα ανέβει στο GitHub)
├── .claude-plugin/
│   └── marketplace.json            ← κατάλογος plugins
├── README.md
└── oc-tools/                       ← το plugin (ομάδα skills)
    ├── .claude-plugin/
    │   └── plugin.json
    └── skills/
        └── file-ownership/
            └── SKILL.md            ← θα κάνω paste το περιεχόμενο
```

---

## 4. Περιεχόμενα αρχείων (templates)

### `.claude-plugin/marketplace.json`
```json
{
  "$schema": "https://anthropic.com/claude-code/marketplace.schema.json",
  "name": "my-oc-skills",
  "description": "Προσωπικά Claude Code skills για OpenCart projects",
  "owner": {
    "name": "digital4u",
    "email": "dev@digital4u.gr"
  },
  "plugins": [
    {
      "name": "oc-tools",
      "description": "Skills & helpers για OpenCart development (ownership, sync, ERP κ.λπ.)",
      "author": { "name": "digital4u" },
      "category": "development",
      "source": {
        "source": "git-subdir",
        "url": "https://github.com/<USERNAME>/my-oc-skills.git",
        "path": "oc-tools",
        "ref": "main"
      }
    }
  ]
}
```
> ⚠️ Άλλαξε το `<USERNAME>` με το πραγματικό GitHub username μόλις δημιουργηθεί το repo.

### `oc-tools/.claude-plugin/plugin.json`
```json
{
  "name": "oc-tools",
  "description": "Skills & helpers για OpenCart development",
  "author": { "name": "digital4u", "email": "dev@digital4u.gr" }
}
```

### `oc-tools/skills/file-ownership/SKILL.md` (placeholder — θα το αντικαταστήσω)
```markdown
---
name: file-ownership
description: <μία γραμμή που περιγράφει πότε να ενεργοποιείται το skill>
---

# file-ownership

<εδώ θα κολλήσω το πραγματικό περιεχόμενο στο νέο session>
```

### `README.md`
```markdown
# my-oc-skills
Προσωπικός Claude Code skills marketplace για OpenCart projects.

## Εγκατάσταση σε ένα μηχάνημα
    claude plugin marketplace add https://github.com/<USERNAME>/my-oc-skills
    claude plugin install oc-tools@my-oc-skills
```

---

## 5. Μορφή ενός `SKILL.md` (για αναφορά όταν φτιάχνω κι άλλα)

```markdown
---
name: kebab-case-όνομα
description: Πότε/γιατί ενεργοποιείται. Σαφές, γιατί βάσει αυτού αποφασίζει το Claude αν θα το φορτώσει.
---

# Τίτλος

Οδηγίες / βήματα / κανόνες που θέλω να ακολουθεί το Claude όταν τρέξει αυτό το skill.
Μπορεί να έχει και βοηθητικά αρχεία στον ίδιο φάκελο (scripts, templates) και να τα αναφέρει.
```

- Ο φάκελος κάθε skill = το `name`. Μέσα ΥΠΟΧΡΕΩΤΙΚΑ ένα `SKILL.md`.
- Το `description` πρέπει να λέει καθαρά «πότε χρησιμοποιείται» — αυτό διαβάζει το Claude.

---

## 6. Βήματα που θέλω να κάνει το Claude στο νέο session

1. Φτιάξε τη δομή φακέλων/αρχείων του §3–§4 (πρότεινε path, π.χ. `/root/my-oc-skills` ή μέσα στο home).
2. Ρώτησέ με να κάνω **paste το πραγματικό περιεχόμενο του `file-ownership/SKILL.md`** και αντικατέστησε το placeholder.
3. `git init`, πρώτο commit. (ΜΗΝ κάνεις push χωρίς να σε το πω — πρώτα θα φτιάξω/δώσω το GitHub repo URL.)
4. Όταν δώσω το URL: συμπλήρωσε το `<USERNAME>` στο `marketplace.json` + `README.md`, commit, και πες μου τις
   ακριβείς εντολές για push (`git remote add origin … && git push -u origin main`).
5. Δώσε μου τις εντολές εγκατάστασης για ΤΟΥΤΟ το μηχάνημα και επιβεβαίωσε με `/reload-skills` ότι φορτώθηκε.

---

## 7. Καθημερινή χρήση μετά το setup

> Πραγματικό repo: **`Opencart4u/claude_skills`** · marketplace name: **`claude_skills`** · plugin: **`oc-tools`**

- **Νέο μηχάνημα/server (μία φορά):** μέσα στο Claude Code prompt
  ```
  /plugin marketplace add Opencart4u/claude_skills
  /plugin install oc-tools@claude_skills
  ```
  (Shell ισοδύναμα για headless/scripts: `claude plugin marketplace add Opencart4u/claude_skills` και `claude plugin install oc-tools@claude_skills`.)
- **Προσθήκη/αλλαγή skill:** edit στο repo → `git push` → στα μηχανήματα `/plugin marketplace update claude_skills`.
- Από εκεί και πέρα φορτώνονται **αυτόματα σε κάθε session**.
- Επιβεβαίωση φόρτωσης: `/reload-plugins` (δείχνει «Reloaded X plugins, 2 skills…») ή `/plugin` → tab **Installed**.

---

## 7b. SSH / πολλά μηχανήματα — τι ισχύει ανά server

**Το install γίνεται μία φορά ανά SERVER, όχι ανά project/session.** Το plugin αποθηκεύεται στο
`~/.claude` του χρήστη του server. Άρα:

| Σενάριο | Χρειάζεται install; |
|---|---|
| Ίδιος server, ξανά SSH / άλλα projects / νέα sessions | ❌ Όχι — μένει στο `~/.claude`, φορτώνει αυτόματα |
| **Καινούργιος** server (πρώτη φορά) | ✅ Ναι, μία φορά τα δύο `/plugin` commands |
| Άλλαξε κάτι στο repo (`git push`) | Στους servers: `/plugin marketplace update claude_skills` |

⚠️ **Εξαίρεση — εφήμεροι servers** (containers/VMs που στήνονται καθαροί κάθε φορά, ή χάνεται το
`~/.claude`): εκεί το install χάνεται και ξαναχρειάζεται. Αυτοματοποίησέ το βάζοντας στο
provisioning / `.bashrc` / dotfiles του server (τρέχουν headless, χωρίς interactive prompt):

```bash
claude plugin marketplace add Opencart4u/claude_skills
claude plugin install oc-tools@claude_skills
```

⚠️ **Auth:** αν το repo `claude_skills` είναι **private**, το `marketplace add` σε νέο server θέλει
git auth (GitHub login / credential manager / SSH key / token). Αν είναι **public**, δεν χρειάζεται
τίποτα — απλούστερο για πολλά μηχανήματα.

---

## 8. Σημειώσεις για το τρέχον project (context, αν χρειαστεί)

- Project: OpenCart 3.0.4.1, `lili.beta4u.gr`, DB prefix `oc3xtbl_`, MySQL 8.0.
- ERP: API4U → SoftOne. Integration controller: `catalog/controller/extension/api4u/integration.php`.
- (Άσχετο με το marketplace — απλώς για συνέχεια αν ρωτήσω.)
```

# Repo Template

Standard-Vorlage für neue Repositories. Enthält die Grundausstattung, die jedes
neue Projekt sowieso bekommen soll, damit sie nicht jedes Mal manuell
nachgezogen werden muss.

> **Sichtbarkeit: öffentlich.** Gehostete Actions-Runner sind hier kostenlos,
> Secret Scanning, Push Protection und CodeQL Default Setup sind verfügbar (Repo-Settings,
> Besitzer-Entscheidung). Das private Schwesterrepo `repo-template` ist sonst identisch.

## Verwendung

1. Auf GitHub bei einem neuen Repo **"Repository template"** auf dieses Repo
   setzen (Dropdown beim Erstellen, oder Button "Use this template" auf dieser
   Repo-Seite). GitHub kopiert dann Dateien und Struktur, aber **nicht**
   Branch-Protection, Secrets oder Repo-Settings — die müssen pro neuem Repo
   einmal nachgezogen werden (siehe unten).
2. `CLAUDE.md` oben ausfüllen (`<PROJECT_NAME>`, Type/Stack/Structure/Commands/
   Notes) — der untere Abschnitt "Working rules" bleibt unverändert, der
   spiegelt die globale Claude-Code-Konfiguration.
3. `NOTICE` und `README.md` an das eigentliche Projekt anpassen.
4. CI einrichten: in `.github/workflows/ci.yml` den Stack-Block des Projekts
   einkommentieren (Job `build` nicht umbenennen) und in
   `.github/dependabot.yml` die genutzten Ecosystems. Secrets, Runner-Wahl,
   Labels, Rulesets und Kosten stehen in
   [`docs/TEMPLATE-SETUP.md`](docs/TEMPLATE-SETUP.md) (englisch, danach löschen).
5. Optional Claude-Review und `@claude`: Secret `CLAUDE_CODE_OAUTH_TOKEN` setzen
   (`gh secret set CLAUDE_CODE_OAUTH_TOKEN --repo <owner>/<neues-repo>`, `gh`
   fragt den Wert ab). Ohne Secret überspringen beide Workflows ihre Schritte
   und bleiben **grün** — "grün" heißt dann "übersprungen", nicht "geprüft".
6. Merge-Gate: Ruleset aus `.github/rulesets/default.json` erst anwenden, wenn die
   Checks auf `main` grün gelaufen sind und der Besitzer die Check-Namen
   bestätigt hat (Anleitung in `.github/rulesets/README.md`). Repo-Settings,
   Secrets und Rulesets ändert nie ein Workflow-PR.

## Was hier drin ist

| Datei | Zweck |
|---|---|
| `CLAUDE.md` | Arbeitsregeln für Claude Code (Git-Workflow, Security, Subagenten) + Platzhalter für Projekt-Kontext |
| `.claude/settings.json` | Permissions, die Claude Code in diesem Repo automatisch erlaubt sind |
| `LICENSE` / `NOTICE` | GNU General Public License v3.0 |
| `.gitignore` / `.editorconfig` | generische Basis-Regeln |
| `CONTRIBUTING.md` | Branch-Namen, Commit-Konvention, PR-Ablauf |
| `SECURITY.md` | Meldeweg für Sicherheitslücken |
| `CHANGELOG.md` | Keep-a-Changelog-Format, startet leer |
| `.github/ISSUE_TEMPLATE/` | Bug-Report + Feature-Request |
| `.github/PULL_REQUEST_TEMPLATE.md` | PR-Checkliste |
| `.github/workflows/claude.yml` | `@claude` in Issues/PR-Kommentaren reagiert (ohne Secret übersprungen) |
| `.github/workflows/claude-code-review.yml` | Claude-Code-Review auf jeden PR, gehärtet; Job `claude-review` (ohne Secret übersprungen) |
| `.github/workflows/ci.yml` | Stack-neutrales Skelett, Job `build` |
| `.github/workflows/workflow-lint.yml` | actionlint + zizmor, Job `workflow-lint` |
| `.github/workflows/secret-scan.yml` | gitleaks über den PR-Diff, Job `gitleaks` (`.gitleaks.toml`) |
| `.github/workflows/labels.yml` + `.github/labels.json` + `scripts/ci/` | Labels als Code, **opt-in** über die Repo-Variable `LABELS_ENABLED` |
| `.github/dependabot.yml` | `github-actions` wöchentlich, 7 Tage Cooldown; Platzhalter für pip/npm/docker |
| `.github/rulesets/` | Ruleset-Vorlage (`default.json`) + Erklärung, wird **nicht** automatisch angewandt |
| `docs/TEMPLATE-SETUP.md` | Einrichtung eines neuen Repos: Secrets, Runner, Rulesets, Kosten |

## Lizenz

GNU General Public License v3.0 — siehe [`LICENSE`](LICENSE). Gilt für alle Projekte, die aus
dieser Vorlage entstehen, sofern im jeweiligen Repo nicht ausdrücklich anders
vermerkt.

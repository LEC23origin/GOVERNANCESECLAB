# LEC23 Terra Classic Governance Sentinel V2

**GitHub-artiger Sicherheitsprüfplatz für Terra Classic, keine echte Governance-Vollmacht.** Alle Entscheidungen in der Anwendung sind zunächst **Off-Chain-Simulationen**. Live-API-Daten und echte Wallet-Signaturen sind gesondert gekennzeichnet. Die von den Tests verwendeten GitHub-/Governance-Antworten waren **Mock-Daten** und werden nicht als aktuelle Chain-Zustände ausgegeben.

## Sofort verwenden

1. Öffne `LEC23_Terra_Governance_Sentinel_V2.html` im Browser. Die bisherigen Anträge, lokalen Regelprüfungen, Audit-Protokolle sowie die neuen Oberflächen funktionieren ohne Installation.
2. **GitHub / CI:** Ein öffentliches Repository im Format `owner/repo` eingeben (vorausgefüllt: `classic-terra/core`) und `PRs laden` anklicken. In der Warteschlange `PR prüfen` auswählen. Die App importiert maximal 100 Dateimetadaten und kürzt größere Diffs; unvollständige Ausschnitte werden ausdrücklich markiert.
3. **Governance Live:** `Live aktualisieren` lädt maximal 30 Vorschläge über erlaubte Terra-Classic-LCD-Endpunkte, prüft `columbus-5` und analysiert *angehängte On-Chain-Messages* anhand von Risikosignalen. `Zur Prüfung übernehmen` importiert diese Messages in einen lokalen Sicherheitsfall.
4. **Prüffälle:** Automatischen Regel-Scan verwenden; anschließend ggf. freiwillig eine KI-Zweitmeinung einholen und die unabhängigen Rollenfreigaben lokal simulieren.
5. **Keplr:** Wallet verbinden und einen konkreten Fall-Hash als ADR-36-Nachweis signieren. **Für die kryptografische Validierung** muss der lokale Node-Server laufen. Ohne ihn bleibt die Signatur gespeichert, aber als **nicht verifiziert** markiert.
6. **Audit-Protokoll:** Lokale Fälle und Nachweise als JSON exportieren. Browser-LocalStorage ist manipulierbar und kein revisionssicheres Log.

## Lokaler Server: kryptografische Nachweisprüfung + optionale KI

Node.js 20+ im Ordner des entpackten ZIP-Pakets starten:

```bash
node LEC23_Sentinel_Server_V2.mjs
```

Öffne `http://127.0.0.1:8787`. Der Server betreibt `/api/verify-signature` zur unabhängigen ADR-36-Prüfung. Er verifiziert SHA-256/ECDSA-secp256k1, die gebundene Terra-Adresse und eine zeitlich begrenzte Review-Nachricht. **Dies authentifiziert nur den Besitz der privaten Wallet-Schlüssel**, nicht die Zugehörigkeit zu Auditoren, Validatoren oder einer Governance-Gruppe. Ein wiederholt vorgelegter Beleg ist keine Login-Session; für echte Logins wären serverseitige Nonces und Replay-Schutz erforderlich.

Für zusätzliche KI-Analyse den API-Schlüssel **nur als Umgebungsvariable des Servers** setzen (niemals in HTML oder GitHub):

```bash
OPENAI_API_KEY="YOUR_KEY" OPENAI_MODEL="gpt-6-luna" node LEC23_Sentinel_Server_V2.mjs
```

Der KI-Prüfbericht läuft über die Responses API. Der Code wird erst nach ausdrücklichem Klick und Bestätigung an den konfigurierten Modellanbieter gesendet. `store:false` ist gesetzt. KI ist beratend; **kein automatisches Deployment, kein Beweis für sicheren Code und keine automatischen Freigaben**. Ohne Schlüssel bleibt die lokale Regelprüfung nutzbar.

Cloudflare Pages kann die **statische HTML** hosten. Die `.mjs`-Node-Anwendung läuft dort **nicht unverändert**; für Keplr-Validierung/KI wird ein separater vertrauenswürdiger Server oder eine angepasste Worker-/Functions-Implementierung benötigt. HTTPS ist in Produktion Pflicht.

## GitHub Actions – automatischer Sicherheitsgate bei jedem Pull Request

Im Zielrepository diese beiden Dateien in der entsprechenden Verzeichnisstruktur anlegen:

- `.github/workflows/sentinel-security.yml`
- `scripts/sentinel-ci-scan.mjs`

Das Workflow-Beispiel läuft bei PRs gegen `main` oder `master` und manuell. Es fordert nur `contents: read`, checkt Quellcode aus, führt **nicht-ausführende Musterprüfungen** über geänderte Dateien durch und veröffentlicht `.sentinel-reports/report.json` als Action-Artefakt. **Kritische Befunde** lassen den Job fehlschlagen; unvollständig lesbare Quellen führen ebenfalls zu einem Fehlstatus. Es werden weder untrusted Build-/Test-Skripte automatisch ausgeführt noch Geheimnisse an PRs weitergereicht. Zusätzliche reproduzierbare Tests, Dependency-Scanning und Sandbox-Läufe sind als nächste Produktionsstufe einzurichten.

Das Workflow-Template muss in **jedem zu schützenden Repository** installiert und als erforderlicher Branch-Protection-Check konfiguriert werden. Andernfalls blockiert ein roter CI-Status keinen Merge automatisch.

## Prüfungen ausführen

```bash
node --test test/*.test.mjs
node scripts/sentinel-ci-scan.mjs
```

Sechs automatisierte Unit-Tests prüfen normale/quarantänepflichtige Muster und ECDSA-ADR-36-Nachweise; zusätzlich wurde die HTML-Oberfläche auf Desktop und Mobil in Browser-Tests mit simulierten API-Antworten geprüft. Es wurden **keine realen KI-Aufrufe, GitHub-Änderungen oder On-Chain-Transaktionen** ausgeführt.

## Sicherheitsmodell und Grenzen

- **Repo und PR-Quellen:** Public GitHub API, ohne Token, read-only. GitHub-Limits, 403/404, CORS und gekürzte API-Antworten werden ausgewiesen. Ein importierter PR ist kein vollständiger Code-Audit. Ein Github-CI-Status ist beobachtete Metadateninformation, kein Beweis für sichere Semantik.
- **On-Chain:** Die App verwendet nur erlaubte LCD-Endpunkte, validiert `columbus-5` und liest Governance-Nachrichten sowie, falls verfügbar, die aktuelle `x/wasm`-Berechtigung. Nie Broadcast, Upload, Abstimmung oder Governance-Regeländerung.
- **Rollen:** Lokale Auditor-/Validator-Rollen und Abstimmungsgewichte bleiben *frei auswählbare Demo-Daten*. Ein ADR-36-Wallet-Nachweis ändert diese Rollen nicht. Eine produktive Rollenvergabe braucht eine on-chain/extern autorisierte, überprüfbare Registratur.
- **Sicherheit:** Static Scanning und KI können Fehler übersehen oder falsch bewerten. Produktionsbetrieb benötigt vollständige Artefakt-/Commit-Bindung, reproduzierbare Builds, echten Sandbox-/Testnet-Lauf, mehrere unabhängige Reviews, Rollen-/Schlüsselmanagement, zeitlich begrenzte Rechte, Logs außerhalb des Browsers, CI-Integrität, Abwehr von API-Spoofing und eine bewilligte Governance-Notfallregel.
- **Notfallspur:** Priorisiert Analyse und Freigabevorbereitung, **umgeht niemals** gültige On-Chain-Regeln. Ein kritischer Befund sperrt den lokalen Vorbereitungsprozess. Durch Proposal #12234 wurden CosmWasm-Code-Uploads an die Governance gebunden; für Sonderbefugnisse braucht es zuerst rechtmäßige Governance-Regeln.

Quellen: 
- https://validator.info/terra-classic/governance/12234
- https://docs.terra-classic.money/develop/endpoints/
- https://docs.github.com/en/rest/pulls/pulls
- https://docs.github.com/en/rest/commits/commits
- https://docs.github.com/en/rest/code-scanning/code-scanning
- https://docs.keplr.app/

**V2 ist ein Forschungsprototyp mit funktionsfähigen Modulen und Tests – kein produktives Sicherheitsprodukt.**

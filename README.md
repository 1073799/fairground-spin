# 🎡 Fairground Spin – Gewichtetes Glücksrad

Ein Glücksrad für Gruppenentscheidungen: Frage eintippen, Antwortoptionen
sammeln und **nach Stimmen gewichten**. Wer 7 Stimmen hat, bekommt ein
entsprechend größeres Tortenstück – das Rad entscheidet fair nach Anteilen.

Mit optionalem **Live-Voting über Supabase**: Alle im selben Raum sehen
Optionen, Stimmen und das Dreh-Ergebnis in Echtzeit.

![Glücksrad](screenshots/hero.png)

---

## ✨ Funktionen

- **Gewichtete Segmente** – Stimmen pro Option per +/− Zähler, das Rad
  passt die Anteile automatisch an.
- **Automatische Emojis** – je nach eingetippter Option (Pizza → 🍕,
  Kino → 🎬). Manuell überschreibbar per Emoji-Picker, neutraler Fallback
  für Unbekanntes.
- **Animierter Dreh** – ratterndes Kleeblatt, pulsierender Knopf, Glühen,
  großes Ergebnis-Popup mit Konfetti und Sound.
- **Verlauf & Admin-Bereich** – jede Entscheidung wird mit Frage, Ergebnis
  und Zeitstempel gespeichert.
- **Teilen per QR-Code** – Link/QR zum Mitmachen.
- **Drei Betriebsstufen** (siehe unten) – von rein lokal bis Echtzeit-Sync.

---

## 🚀 Schnellstart (lokal, ohne Datenbank)

1. Repository herunterladen oder klonen.
2. `index.html` im Browser öffnen – fertig.

In diesem Modus läuft alles lokal im Browser (Stufe 1). Der Verlauf wird
pro Browser gespeichert.

---

## 🧩 Die drei Betriebsstufen

| Stufe | Modus | Verhalten |
|------|-------|-----------|
| **1** | **Lokal** (Fallback) | Ohne Supabase-Konfiguration. Alles läuft im Browser, kein geteiltes Voting. Verlauf liegt lokal (`localStorage`). |
| **2** | **Gleicher Raum** | Sobald Supabase eingerichtet ist, teilen sich alle, die **dieselbe URL** öffnen, automatisch den Raum `main`. Keine zusätzliche Technik nötig. |
| **3** | **Eigene Räume / Live** | Über „Teilen → + Neuer Raum" entsteht ein privater Raum (`?room=ABC12`). QR/Link teilen, alle stimmen in Echtzeit gemeinsam ab. |

Stufe 2 und 3 nutzen dieselbe Technik – der einzige Unterschied ist der
Raum-Code in der URL.

---

## 🔌 Live-Voting einrichten (Supabase)

### 1. Supabase-Projekt anlegen

1. Auf [supabase.com](https://supabase.com) kostenlos registrieren und ein
   neues Projekt erstellen.
2. Unter **Project Settings → API** findest du:
   - **Project URL** (z. B. `https://abcdefgh.supabase.co`)
   - **anon public** Key

### 2. Datenbank-Tabellen anlegen

Im Supabase-Dashboard unter **SQL Editor** dieses Skript ausführen:

```sql
-- Räume: pro Raum eine Zeile mit Frage + Optionen
create table public.rooms (
  id          text primary key,
  question    text  default '',
  options     jsonb default '[]'::jsonb,
  updated_at  timestamptz default now()
);

-- Ergebnisse: jede Dreh-Entscheidung (= Verlauf)
create table public.results (
  id          bigint generated always as identity primary key,
  room_id     text references public.rooms(id) on delete cascade,
  question    text,
  label       text,
  emoji       text,
  color       text,
  created_at  timestamptz default now()
);

-- Realtime aktivieren
alter publication supabase_realtime add table public.rooms;
alter publication supabase_realtime add table public.results;

-- Zugriff (offen, für einfaches Teilen ohne Login)
alter table public.rooms   enable row level security;
alter table public.results enable row level security;

create policy "rooms lesen"     on public.rooms   for select using (true);
create policy "rooms schreiben" on public.rooms   for all    using (true) with check (true);
create policy "results lesen"   on public.results for select using (true);
create policy "results anlegen" on public.results for insert with check (true);
create policy "results löschen" on public.results for delete using (true);
```

> **Hinweis zur Sicherheit:** Die Policies sind bewusst offen, damit jede:r
> ohne Login mitmachen kann. Der anon-Key darf öffentlich sein. Für
> sensible Einsätze solltest du den Zugriff einschränken (z. B. Auth oder
> engere Policies).

### 3. Schlüssel in `index.html` eintragen

Im oberen Bereich von `index.html` (bzw. `Gluecksrad.dc.html`) den
Konfigurationsblock ausfüllen:

```js
window.FAIRGROUND_CONFIG = {
  url: "https://abcdefgh.supabase.co",   // deine Project URL
  anonKey: "eyJhbGciOi..."               // dein anon public Key
};
```

Leere Felder = automatisch lokaler Modus (Stufe 1).

Beim nächsten Laden zeigt das Rad oben **„🟢 Live · Raum main"** statt
„Lokaler Modus".

---

## 🌐 Auf GitHub Pages veröffentlichen

1. Neues Repository **`fairground-spin`** auf GitHub anlegen.
2. `index.html`, `README.md` und den `screenshots/`-Ordner hochladen
   (committen & pushen).
3. Im Repo: **Settings → Pages**.
4. Unter **Build and deployment → Source** „**Deploy from a branch**"
   wählen, Branch `main`, Ordner `/ (root)`, **Save**.
5. Nach kurzer Wartezeit ist das Rad erreichbar unter:
   `https://<dein-github-name>.github.io/fairground-spin/`

Diese URL kannst du teilen – wer sie öffnet, landet (bei eingerichtetem
Supabase) automatisch im gemeinsamen Raum `main`.

---

## 📲 Nutzung

1. **Frage** oben eintippen (z. B. „Wann gehen wir ins Freibad?").
2. **Optionen** anlegen, umbenennen, Emoji wählen.
3. **Stimmen** per **+ / −** gewichten.
4. **DREHEN** – das Rad entscheidet nach Anteilen.
5. **Teilen** (oben links) öffnet QR-Code & Link. Für einen privaten Raum
   „**+ Neuer Raum**".
6. **Admin** (oben rechts) zeigt den gespeicherten Verlauf aller
   Entscheidungen; dort lässt er sich auch löschen.

---

## ⚙️ Projektdateien

| Datei | Zweck |
|-------|-------|
| `index.html` | **Fertige, eigenständige App** (alles eingebettet) – diese Datei wird veröffentlicht. |
| `Gluecksrad.dc.html` | Quell-/Editierfassung (Design Component). |
| `support.js` | Laufzeit-Helfer für die Quellfassung. |
| `README.md` | Diese Anleitung. |

> Zum Anpassen die Quellfassung `Gluecksrad.dc.html` bearbeiten und neu
> bündeln. Für den reinen Betrieb genügt `index.html`.

---

## ⚠️ Gut zu wissen

- **Kein Login nötig** – Räume sind über den Code in der URL „geschützt".
  Wer den Link/QR hat, kann mitmachen.
- **Gleichzeitige Klicks:** Stimmen werden geteilt gezählt; bei exakt
  zeitgleichen Klicks auf dieselbe Option kann in seltenen Fällen eine
  Stimme verloren gehen. Für lockere Gruppenentscheidungen unproblematisch.
- **Offline:** Der lokale Modus funktioniert ohne Internet; Live-Voting
  braucht eine Verbindung zu Supabase.
- **Emoji-Automatik** deckt einen großen, aber endlichen Wortschatz ab –
  Unbekanntes bekommt ein neutrales Symbol, das du manuell ändern kannst.

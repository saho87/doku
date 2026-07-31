# TLS und Verschlüsselung

> **Zielgruppe:** DevOps Engineers, Kubernetes-/OpenShift-Administratoren und technisch Interessierte.
>
> Ziel dieser Dokumentation ist es, die grundlegenden Konzepte moderner Verschlüsselung und TLS zu verstehen – nicht einzelne Befehle oder Produkte.

---

# Inhaltsverzeichnis

- [1. Das eigentliche Problem](#1-das-eigentliche-problem)
- [2. Die Schutzziele der Kryptografie](#2-die-schutzziele-der-kryptografie)
- [3. Symmetrische Verschlüsselung](#3-symmetrische-verschlüsselung)
- [4. Das Schlüsselproblem](#4-das-schlüsselproblem)
- [5. Asymmetrische Verschlüsselung](#5-asymmetrische-verschlüsselung)
- [6. Wie alles zusammenhängt](#6-wie-alles-zusammenhängt)

---

# 1. Das eigentliche Problem

Stell dir vor, dein Browser verbindet sich mit

```
https://www.bank.de
```

Woher weiß dein Browser eigentlich, dass am anderen Ende wirklich die Bank sitzt?

Und woher stammt überhaupt der Schlüssel, mit dem später die gesamte Kommunikation verschlüsselt wird?

Diese beiden Fragen bilden die Grundlage moderner Kryptografie.

Fast alle heute eingesetzten Verfahren – HTTPS, Kubernetes API, OpenShift, GitLab, Vault oder SSH – lösen im Kern genau diese beiden Probleme.

```text
                    Internet

+---------+                            +-------------+
| Browser | -------------------------> |   Server    |
+---------+                            +-------------+

Offene Fragen:

1. Wer ist der Server wirklich?
2. Wie entsteht ein gemeinsamer Schlüssel?
3. Wie verhindern wir das Mitlesen?
```

Bevor wir TLS verstehen können, müssen wir zunächst verstehen, welche Probleme überhaupt gelöst werden müssen.

---

# 2. Die Schutzziele der Kryptografie

Kryptografie verfolgt nicht nur das Ziel, Daten zu verschlüsseln.

Tatsächlich existieren vier grundlegende Schutzziele.

| Schutzziel | Bedeutung |
|------------|-----------|
| Vertraulichkeit | Niemand außer den Kommunikationspartnern kann den Inhalt lesen. |
| Integrität | Daten wurden unterwegs nicht verändert. |
| Authentizität | Die Identität des Kommunikationspartners ist nachweisbar. |
| Nichtabstreitbarkeit | Der Absender kann seine Aktion später nicht glaubhaft bestreiten. |

Diese Ziele werden nicht von einem einzelnen Verfahren erreicht.

Stattdessen arbeiten mehrere kryptografische Verfahren zusammen.

```text
                    Kryptografie

                         │
     ┌───────────────────┼────────────────────┐
     │                   │                    │
     ▼                   ▼                    ▼

Vertraulichkeit     Integrität        Authentizität
      │                  │                  │
      ▼                  ▼                  ▼

Verschlüsselung      Hashfunktionen   Digitale Signaturen
```

TLS kombiniert später genau diese Bausteine zu einem Gesamtsystem.

---

# 3. Symmetrische Verschlüsselung

Die einfachste Form der Verschlüsselung verwendet **einen gemeinsamen Schlüssel**.

Sender und Empfänger besitzen exakt denselben geheimen Schlüssel.

Mit diesem Schlüssel werden Daten verschlüsselt und wieder entschlüsselt.

```text
               Gemeinsamer Schlüssel

             +-----------------------+
             |    Secret Key         |
             +-----------------------+
                  ▲             ▲
                  │             │
                  │             │
+---------+       │             │      +---------+
| Client  |-------+             +------| Server  |
+---------+                      +---------+

        verschlüsseln        entschlüsseln
```

Der große Vorteil dieses Verfahrens ist seine Geschwindigkeit.

Moderne Algorithmen wie AES können selbst mehrere Gigabit pro Sekunde verschlüsseln.

Aus diesem Grund wird nahezu der gesamte Netzwerkverkehr im Internet symmetrisch verschlüsselt.

## Vorteile

- sehr schnell
- geringer Rechenaufwand
- ideal für große Datenmengen

## Nachteil

Beide Kommunikationspartner benötigen **vor Beginn der Kommunikation denselben geheimen Schlüssel**.

Und genau hier beginnt das eigentliche Problem.

---

# 4. Das Schlüsselproblem

Angenommen, zwei Personen möchten erstmals sicher miteinander kommunizieren.

Beide besitzen noch keinen gemeinsamen Schlüssel.

Wie gelangt dieser Schlüssel nun sicher zum Kommunikationspartner?

```text
Client                               Server

     ??? Secret Key ???

──────────── Internet ──────────────

Jeder, der den Schlüssel unterwegs
kopiert, kann anschließend alle
Nachrichten entschlüsseln.
```

Das bedeutet:

Die symmetrische Verschlüsselung ist zwar hervorragend geeignet, **setzt aber bereits einen gemeinsamen geheimen Schlüssel voraus**.

Dieser Schlüssel muss also zunächst auf einem anderen Weg sicher ausgetauscht werden.

Dieses Dilemma wird als **Schlüsselverteilungsproblem** bezeichnet.

Und genau deshalb wurde die asymmetrische Kryptografie entwickelt.

---

# 5. Was wir bisher gelernt haben

Bis jetzt kennen wir nur den ersten Teil der Geschichte.

```text
                 Sichere Kommunikation

                         │
                         ▼

             Symmetrische Verschlüsselung

                         │
                         ▼

             Sehr schnell und effizient

                         │
                         ▼

             Benötigt gemeinsamen Schlüssel

                         │
                         ▼

                Schlüssel existiert nicht

                         │
                         ▼

                  ❓ Problem ungelöst
```

Im nächsten Kapitel lösen wir dieses Problem mithilfe der **asymmetrischen Verschlüsselung**. Dort lernen wir Public Keys, Private Keys und den eigentlichen Durchbruch kennen, der HTTPS und TLS überhaupt erst möglich gemacht hat.

# 6. Asymmetrische Verschlüsselung

Im vorherigen Kapitel haben wir ein fundamentales Problem kennengelernt.

Die symmetrische Verschlüsselung funktioniert hervorragend – **wenn beide Seiten bereits denselben geheimen Schlüssel besitzen.**

Doch genau dieser Schlüssel muss zunächst sicher übertragen werden.

Wie kann man einen geheimen Schlüssel austauschen, ohne dass ihn unterwegs jemand mitliest?

Viele Jahre galt dieses Problem als praktisch unlösbar.

Die Lösung war ein völlig neues Konzept:

**Die asymmetrische Verschlüsselung.**

---

## Eine revolutionäre Idee

Anstatt einen einzigen geheimen Schlüssel zu verwenden, besitzt nun jeder Kommunikationspartner **zwei mathematisch zusammengehörige Schlüssel**.

```text
                Schlüsselpaar

          +----------------------+
          |      Public Key      |
          +----------------------+
                     ▲
                     │
      Gehört zusammen│
                     ▼
          +----------------------+
          |     Private Key      |
          +----------------------+
```

Beide Schlüssel werden gemeinsam erzeugt.

Sie gehören immer zusammen.

Aus dem Public Key lässt sich der Private Key jedoch praktisch nicht berechnen.

Dadurch entsteht eine völlig neue Möglichkeit der sicheren Kommunikation.

---

## Die Rollen der beiden Schlüssel

Die Namen verraten bereits ihre Aufgabe.

### Public Key

Der Public Key darf beliebig verteilt werden.

Er ist **nicht geheim**.

Jeder darf ihn besitzen.

```text
                Public Key

             Server

                │
                │ veröffentlicht
                ▼

      Browser A

      Browser B

      Browser C

      Browser D
```

Der Public Key darf sogar auf einer öffentlichen Webseite stehen.

Seine Veröffentlichung stellt **kein Sicherheitsproblem** dar.

---

### Private Key

Der Private Key ist das genaue Gegenteil.

Er verbleibt ausschließlich beim Besitzer.

```text
               Private Key

            +-------------+
            |   Server    |
            +-------------+

        Darf niemals
        weitergegeben werden.
```

Verliert der Besitzer seinen Private Key, verliert er seine Identität.

Deshalb wird dieser Schlüssel besonders geschützt.

---

## Warum benötigt man überhaupt zwei Schlüssel?

Der eigentliche Durchbruch besteht darin, dass **verschiedene Aufgaben auf unterschiedliche Schlüssel verteilt werden können**.

Der Public Key darf veröffentlicht werden.

Der Private Key bleibt geheim.

Dadurch muss kein geheimer Schlüssel mehr über das Netzwerk übertragen werden.

Das löst erstmals das Schlüsselverteilungsproblem.

```text
                    Früher

Client ---------------- Server

        Secret Key

muss geheim übertragen werden

❌ Problem


──────────────────────────────────────


                    Heute

Client <------------ Public Key

(Server veröffentlicht ihn)


Private Key

bleibt ausschließlich
beim Server

✔ Kein geheimer Schlüssel wird verteilt
```

---

## Moment ...

An dieser Stelle könnte man auf die Idee kommen:

> "Dann verschlüsseln wir doch einfach sämtliche Daten mit dem Public Key."

Das klingt zunächst logisch.

Leider funktioniert das in der Praxis nicht.

Asymmetrische Verschlüsselung ist um Größenordnungen langsamer als symmetrische Verfahren.

Bereits wenige Megabyte würden deutlich mehr Rechenleistung benötigen als dieselben Daten mit AES.

Deshalb eignet sie sich **nicht** für die eigentliche Datenübertragung.

---

## Der eigentliche Durchbruch

Die asymmetrische Verschlüsselung ersetzt die symmetrische Verschlüsselung **nicht**.

Sie ergänzt sie.

```text
          Asymmetrische Kryptografie

                    │
                    ▼

      Sicherer Austausch eines
      gemeinsamen Geheimnisses

                    │
                    ▼

          Symmetrische Kryptografie

                    │
                    ▼

     Schnelle Verschlüsselung aller Daten
```

Genau diese Kombination bildet später das Fundament von TLS.

Asymmetrische Verfahren lösen das Schlüsselproblem.

Symmetrische Verfahren übernehmen anschließend die eigentliche Kommunikation.

---

## Ein neues Problem entsteht

Der Public Key darf öffentlich verteilt werden.

Doch woher weiß der Client eigentlich, dass der empfangene Public Key wirklich zum gewünschten Server gehört?

```text
              Client

                 │
                 ▼

      "Hier ist mein Public Key."

                 ▲
                 │

         Aber von wem?

      Vom echten Server?

      Oder von einem Angreifer?
```

Wir haben also das Schlüsselproblem gelöst.

Dafür ist ein neues Problem entstanden:

**Wie kann ein Client einem Public Key vertrauen?**

Genau dieses Problem lösen wir im nächsten Kapitel mit **digitalen Signaturen und Zertifikaten**.
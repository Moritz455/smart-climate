# Architektur und algorithmische Orchestrierung multimodaler Heiz- und Kühlsysteme in Home Assistant

Die hochintegrierte Gebäudeautomation stellt bei der Klimatisierung von Wohnräumen eine komplexe regelungstechnische Herausforderung dar. Wenn ein Gebäude über bivalente oder trivalente Temperatursteuerungssysteme verfügt – wie im vorliegenden Fall eine Erdwärmepumpe für die Fußbodenheizung, eine Luft-Luft-Wärmepumpe (Split-Klimaanlage) und die Möglichkeit der natürlichen Fensterlüftung –, reicht ein einfacher Zweipunktregler nicht mehr aus. Das primäre Ziel eines fortschrittlichen Smart-Home-Systems unter Home Assistant ist es, diese unterschiedlichen Aktoren nicht nur isoliert zu betreiben, sondern sie in einer Meta-Ebene intelligent zu orchestrieren.

Der Nutzer fordert ein System, das durch die einfache Vorgabe einer Zieltemperatur in einem einzigen digitalen Thermostat im Dashboard selbstständig entscheidet, welche Methode zum Heizen oder Kühlen verwendet wird. Die Entscheidungsparameter sind hierbei maximale Energieeffizienz und Reaktionsschnelligkeit. Dieser Bericht analysiert die thermodynamischen Grundlagen der genannten Systeme, evaluiert verfügbare Home Assistant-Integrationen (HACS und Blueprints) und definiert eine umfassende, maßgeschneiderte Logikarchitektur inklusive detailliertem Pseudo-Code zur Lösung dieser spezifischen Problemstellung.

## Thermodynamische Analyse und Aktor-Charakterisierung

Um eine Entscheidungslogik zu entwerfen, die präzise zwischen Energieeffizienz und Aufheiz- bzw. Abkühlgeschwindigkeit abwägt, müssen die physikalischen Eigenschaften der zur Verfügung stehenden Systeme detailliert charakterisiert werden. Die Systeme unterscheiden sich fundamental in ihrer thermischen Trägheit, der Art der Wärmeübertragung (Konvektion gegenüber Strahlung) und ihrem energetischen Wirkungsgrad, ausgedrückt durch den Coefficient of Performance (COP) beziehungsweise die Energy Efficiency Ratio (EER).

Die Erdwärmepumpe, welche eine Fußbodenheizung speist, fungiert aufgrund der relativ konstanten Quelltemperatur des Erdreichs als das effizienteste System zur Wärmebereitstellung. Die Wärmeabgabe erfolgt primär über langwellige Infrarotstrahlung, was vom menschlichen Körper als besonders angenehm und behaglich empfunden wird. Die signifikante Herausforderung dieses Systems liegt jedoch in der enormen thermischen Masse des Heizestrichs, die zu extremen Totzeiten in der Regelstrecke führt. Wird ein ausgekühlter Raum ausschließlich über die Fußbodenheizung erwärmt, vergehen in der Regel mehrere Stunden, bis die gewünschte Solltemperatur erreicht ist. Zudem neigen solche Systeme ohne prädiktive Regelung zum Überschwingen (Overshoot), da der erhitzte Estrich auch nach Abschaltung der Umwälzpumpe weiterhin Wärme an den Raum abgibt.

Im Gegensatz dazu steht die Split-Klimaanlage, die als Luft-Luft-Wärmepumpe fungiert. Sie führt dem Raum Wärme oder Kälte direkt über erzwungene Konvektion zu. Da die Raumluft eine vergleichsweise geringe spezifische Wärmekapazität besitzt, lassen sich Zieltemperaturen in kürzester Zeit (wenige Minuten) realisieren. Der COP einer Luft-Luft-Wärmepumpe ist im Heizbetrieb stark von der Außentemperatur abhängig und fällt bei sehr niedrigen Außentemperaturen ab, während die Erdwärmepumpe stabile Arbeitszahlen liefert. Der entscheidende Vorteil der Split-Klimaanlage liegt somit in ihrer unübertroffenen Agilität, was sie zum idealen Aktor für schnelle Temperaturkorrekturen macht.

Die dritte Methode, die natürliche Fensterlüftung, stellt im Kühlfall die effizienteste Option dar, da sie – abgesehen von menschlicher Interaktion oder motorisierten Fensteröffnern – keine elektrische Energie für einen thermodynamischen Kreisprozess benötigt (der theoretische COP strebt gegen unendlich). Die Effektivität dieses Systems hängt jedoch vollständig von den meteorologischen Randbedingungen ab und birgt ohne sensorische Überwachung das Risiko bauphysikalischer Schäden.

| System                           | Primäre Funktion | Art der Wärmeübertragung | Reaktionszeit           | Thermische Trägheit      | Effizienz (Heizen/Kühlen)          | Spezifische Charakteristik und Einsatzzweck                                                                             |
| -------------------------------- | ---------------- | ------------------------ | ----------------------- | ------------------------ | ---------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| Fußbodenheizung (Erdwärmepumpe)  | Heizen           | Strahlung (primär)       | Sehr langsam (2–6 h)    | Sehr hoch (Estrichmasse) | Sehr hoch (COP > 4.5, quellstabil) | Nutzt die konstante Temperatur des Erdreichs; exzellent zur Erhaltung der Solltemperatur und Deckung der Grundlast.     |
| Split-Klimaanlage (Luft-Luft-WP) | Heizen & Kühlen  | Erzwungene Konvektion    | Sehr schnell (5–15 min) | Sehr gering (Raumluft)   | Hoch (COP ca. 3.0 – 4.0, variiert) | Erwärmt oder kühlt direkt die Luftmassen; ideal für Spitzenlasten und extrem schnelle Temperaturkorrekturen.            |
| Natürliche Fensterlüftung        | Kühlen           | Luftaustausch            | Mittel bis schnell      | Gering                   | Unendlich (Kosten = 0)             | Höchste Energieeffizienz beim Kühlen, jedoch stark abhängig von der Außentemperatur und den Feuchtigkeitsverhältnissen. |

Das grundlegende Spannungsfeld dieser Architektur ist der Konflikt zwischen Effizienz und Geschwindigkeit. Die Automatisierung in Home Assistant muss dynamisch evaluieren, wie groß die Diskrepanz zwischen der Ist-Temperatur und der Ziel-Temperatur ist, um zu entscheiden, ob die träge, aber hocheffiziente Grundlast (Erdwärme) ausreicht, oder ob die agile, aber konvektive Spitzenlast (Split-Klima) zugeschaltet werden muss.

## Sensorik und bauphysikalische Limitationen

Eine präzise Regelung erfordert verlässliche Eingangsgrößen. Der Anwender spezifiziert, dass aktuell lediglich die Innentemperatur und die Außentemperatur als Sensordaten zur Verfügung stehen. Für den reinen Heizbetrieb ist diese Datengrundlage absolut ausreichend. Die Differenz zwischen der vom Nutzer eingestellten Solltemperatur und der gemessenen Raumtemperatur (das Temperatur-Delta $\Delta T$) liefert den primären Indikator für den Wärmebedarf, während die Außentemperatur in erweiterten Konzepten für eine Heizkurvenanpassung der Wärmepumpe herangezogen werden kann.

Kritisch wird diese sensorische Limitation jedoch im Kühlbetrieb, insbesondere wenn die Entscheidung zur natürlichen Fensterlüftung getroffen werden soll. Eine rein temperaturbasierte Entscheidung – das Fenster zu öffnen, weil die Außentemperatur unter der Innentemperatur liegt – ist bauphysikalisch riskant. Die Temperatur allein gibt keine Auskunft über den absoluten Wassergehalt der Luftmassen. Wenn es im Sommer nach einem Gewitter draußen abkühlt, die Luft jedoch zu nahezu 100 Prozent gesättigt ist, strömt beim Lüften eine massive Menge an Feuchtigkeit in den Raum. Trifft diese schwüle Luft auf die durch die Erdwärmepumpe (im passiven Kühlbetrieb) oder die Split-Klimaanlage eventuell abgekühlten Bauteile, kann die Temperatur lokal unter den Taupunkt fallen, was zu Kondensatbildung und mittelfristig zu gefährlichem Schimmelwachstum führt.

Aus professioneller Sicht wird daher dringend angeraten, das Sensornetzwerk um präzise Hygrometer (Luftfeuchtigkeitssensoren) für den Innen- und Außenbereich zu erweitern. Dies ermöglicht die Berechnung der absoluten Feuchtigkeit in Gramm Wasser pro Kubikmeter Luft ($g/m^3$) sowie des Taupunkts. Erst der direkte Vergleich der absoluten Feuchtigkeitswerte offenbart, ob ein Luftaustausch den Raum tatsächlich trocknet oder befeuchtet. Der algorithmische Ablaufplan (Pseudo-Code) am Ende dieses Berichts wird primär auf die vorhandenen Temperatursensoren abgestimmt sein, um der initialen Nutzeranforderung gerecht zu werden. Die theoretische Erweiterung um die absolute Feuchtigkeit wird jedoch als essenzielles Modul für künftige Ausbaustufen skizziert.

## Evaluation des Home Assistant Ökosystems (HACS und Blueprints)

Um die Orchestrierung in Home Assistant umzusetzen, muss zunächst das bestehende Ökosystem aus Kernfunktionen, Blueprints und den Integrationen aus dem Home Assistant Community Store (HACS) auf seine Eignung für diese spezifische Aufgabenstellung geprüft werden. Die Recherche zeigt deutlich, dass keine einzelne "Out-of-the-Box"-Lösung existiert, die ein solches multimodales, hybrides System vollautomatisch nach dynamischen Effizienz- und Geschwindigkeitskriterien regelt. Die vorhandenen Lösungen bieten jedoch herausragende Teilsysteme, die als Bausteine für eine übergeordnete Architektur dienen können.

### Versatile Thermostat (HACS)

Der "Versatile Thermostat" ist eine der fortschrittlichsten und umfassendsten Thermostat-Integrationen, die derzeit für Home Assistant verfügbar sind. Sein primäres Alleinstellungsmerkmal ist die Implementierung eines hochentwickelten TPI-Algorithmus (Time Proportional and Integral), der speziell dafür konzipiert wurde, träge Heizsysteme wie Fußbodenheizungen zu bändigen. Normale Zweipunktregler schalten die Heizung ein, bis die Zieltemperatur erreicht ist, und dann aus, was bei Flächenheizungen zu massivem Überschwingen führt. Der TPI-Algorithmus des Versatile Thermostat moduliert die Einschaltzeit des Heizsystems proportional zur verbleibenden Temperaturdifferenz, wodurch die Zieltemperatur asymptotisch und ohne Überschwingen angefahren wird.

Zudem bietet die Integration ein tiefes Funktionsportfolio wie die automatische Fenster-offen-Erkennung durch Temperaturabfall-Messung, Präsenzerkennung, vordefinierte Presets (Eco, Comfort, Boost) und ein intelligentes Power-Shedding zur Vermeidung von Netzüberlastungen. Auch das direkte Überschreiben bestehender Climate-Entitäten (mittels der Funktion over_climate) ist möglich.

Trotz dieser Mächtigkeit scheitert der Versatile Thermostat als alleinige Lösung für die vorliegende Aufgabenstellung. Er ist konzeptionell darauf ausgelegt, exakt ein primäres Heizsystem pro Instanz zu optimieren. Die fluide, proaktive und dynamische Umschaltung zwischen zwei physikalisch völlig gegensätzlichen Systemen – der Strahlungswärme des Bodens und der Konvektionswärme der Split-Klimaanlage – basierend auf der aktuellen Aufheizgeschwindigkeit, ist nativ nicht abgebildet. Er eignet sich jedoch exzellent als Sub-Controller für die Fußbodenheizung innerhalb einer größeren Architektur.

### Dual Smart Thermostat (HACS)

Eine weitere populäre Alternative ist der "Dual Smart Thermostat", der den nativen, generischen Thermostat von Home Assistant um wesentliche Funktionen erweitert. Seine Kernkompetenz liegt in der Fähigkeit, sowohl Heiz- als auch Kühlgeräte innerhalb einer einzigen Thermostat-Karte (Modus heat_cool) zu verwalten und entsprechende Zieltemperatur-Bänder (target_temp_low und target_temp_high) bereitzustellen.

Besonders interessant für hybride Systeme ist die implementierte "Two Stage (AUX) Heating"-Funktion. Hierbei kann ein sekundäres, unterstützendes Heizgerät definiert werden. Die Zuschaltung dieses Hilfssystems erfolgt jedoch streng deterministisch und zeitbasiert. Erreicht das primäre System nach Ablauf eines fest definierten Timeouts die Solltemperatur nicht, wird das sekundäre System aktiviert. Für den hier geforderten Anwendungsfall ist diese Logik zu starr. Ein simples Timeout ignoriert die thermodynamische Effizienz und die aktuellen Umweltbedingungen. Wenn ein Raum stark ausgekühlt ist, muss das schnelle Konvektionssystem sofort anlaufen und nicht erst nach Ablauf eines stundenlangen Timeouts der Fußbodenheizung. Ebenso fehlt dieser Integration jegliche Logik zur Evaluierung einer natürlichen Fensterlüftung.

### Thermal Comfort (HACS)

Auch wenn dem Nutzer derzeit nur Temperatursensoren zur Verfügung stehen, ist die Integration "Thermal Comfort" aus technologischer Sicht für das Lüftungsmanagement unerlässlich. Sobald das System um Hygrometer erweitert wird, berechnet diese Komponente im Hintergrund deterministisch den Taupunkt und die absolute Feuchtigkeit (in $g/m^3$) aus den Rohdaten für Temperatur und relativer Feuchte.

Wie in der bauphysikalischen Analyse dargelegt, ist der Vergleich der absoluten Feuchtigkeit zwischen Innen- und Außenluft der einzig valide Parameter, um zu entscheiden, ob eine Fensterlüftung das Raumklima verbessert oder verschlechtert. Home Assistant Automatisierungen können direkt auf diese berechneten Werte zugreifen und somit verhindern, dass Lüftungsempfehlungen ausgesprochen werden, wenn die Außenluft zwar kühler, aber absolut feuchter ist als die Raumluft.

### Blueprints zur Lüftungsempfehlung

In der Home Assistant Community kursieren diverse Blueprints, wie beispielsweise die "Lüftungsempfehlung v0.4.1" oder Konzepte zur "Advanced Heating Control". Diese Skripte bieten häufig fertige Benachrichtigungsketten für das Öffnen von Fenstern bei optimalen Außenbedingungen. Sie stellen eine erhebliche Erleichterung dar, wenn es lediglich um eine Push-Nachricht geht. Sie lassen sich jedoch nicht in eine geschlossene, bivalente Heiz- und Kühlregelung integrieren, in der das Lüften des Fensters in direkter algorithmischer Konkurrenz zum Anspringen der Split-Klimaanlage steht. Für eine echte Orchestrierung muss die Logik maßgeschneidert werden.

### Climate Template (HACS)

Da keines der fertigen Thermostate die logische Entscheidungsfindung übernehmen kann, bedarf es einer Komponente, die dem Nutzer im Dashboard ein reguläres Thermostat präsentiert, im Hintergrund jedoch keine fest verdrahteten Relais schaltet, sondern komplexe Skripte ausführt. Hierfür ist das "Climate Template" die absolute Ideallösung.

Diese Integration ermöglicht es, einen völlig freien, virtuellen Thermostat zu erschaffen. Anstatt auf einen Schalter zu wirken, können bei Statusänderungen (wie der Änderung der Zieltemperatur oder dem Wechsel von Heizen auf Kühlen) beliebige Home Assistant Automatisierungen oder Skripte aufgerufen werden. Dies entkoppelt die Präsentationsschicht (das UI, in dem der Nutzer seine Wunschtemperatur einstellt) vollständig von der Daten- und Logikschicht, in der die mathematische Evaluierung der effizientesten Heiz- oder Kühlmethode stattfindet.

## Architektur der multimodalen Orchestrierung

Die Synthese der bisherigen Analysen führt zu einer klaren Architektur für die Umsetzung in Home Assistant. Das System muss in drei strikt getrennte Schichten unterteilt werden, um Wartbarkeit, Stabilität und Erweiterbarkeit zu garantieren.

Die oberste Schicht bildet die Präsentationsschicht. Hier interagiert der Nutzer mit dem System. Realisiert wird dies über einen virtuellen Master-Thermostat, der mittels der climate_template-Integration konfiguriert wird. Der Nutzer sieht lediglich die aktuelle Raumtemperatur und stellt seine gewünschte Solltemperatur sowie den globalen Betriebsmodus (Auto, Heizen, Kühlen, Aus) ein. Es findet keine direkte Interaktion mit der Wärmepumpe oder der Klimaanlage statt.

Darunter liegt die Daten- und Berechnungsschicht. Hier fließen die Sensordaten (Innen- und Außentemperatur) zusammen. Für künftige Ausbaustufen würde hier auch die "Thermal Comfort"-Integration verortet sein, um die absolute Feuchtigkeit zu berechnen. Zudem wird in dieser Schicht der Status der Fenstersensoren (sofern vorhanden) erfasst, um zu prüfen, ob der Nutzer einer Lüftungsempfehlung nachgekommen ist.

Das Herzstück bildet die Logik- und Orchestrierungsschicht. Ein Python-Skript (über PyScript/AppDaemon) oder eine komplexe native Home Assistant Automation wertet periodisch oder bei Statusänderungen der Präsentationsschicht die Daten aus. Diese Schicht berechnet das Temperatur-Delta ($\Delta T$) zwischen Soll- und Ist-Wert und entscheidet anhand vordefinierter Schwellenwerte, welche Aktoren in der Ausführungsschicht (der Erdwärmepumpe und der Split-Klimaanlage) aktiviert werden.

### Single-Room-Regelung vs. Gesamtgebäude-Zonierung

Der Nutzer fragt nach einer Lösung für "einen einzelnen Raum oder das gesamte Haus". Hierbei muss regelungstechnisch scharf differenziert werden. Eine Einzelraumregelung (Zonierung) setzt voraus, dass die Heizkreise der Fußbodenheizung über thermoelektrische Stellantriebe an den Heizkreisverteilern separat steuerbar sind. Die Orchestrierungslogik wird in diesem Fall pro Raum instanziiert. Jeder Raum erhält seinen eigenen virtuellen Master-Thermostat, der die Stellantriebe des Raumes sowie die lokale Split-Klimaanlage koordiniert.

Wird das System hingegen für das gesamte Haus ausgelegt, fungiert ein zentraler Referenzraum (meist das Wohnzimmer) als Taktgeber, oder es wird ein Mittelwert aller Raumsensoren gebildet. Die Logik greift dann direkt auf den Vorlauf oder den zentralen Betriebsmodus der Erdwärmepumpe zu. Letzteres ist bei modernen Erdwärmepumpen mit hervorragendem hydraulischem Abgleich energetisch oft effizienter, da thermoelektrische Stellantriebe, die den Durchfluss abrupt kappen, die Hydraulik der Wärmepumpe stören können. In der folgenden Logikbetrachtung wird von einer logischen Entität für die Heizung (climate.erdwaermepumpe) und einer für die Klimaanlage (climate.split_klima) ausgegangen, was sowohl einen Raum als auch ein ganzes Haus repräsentieren kann.

## Entwicklungslogik und Regelalgorithmen

Die Orchestrierung der Aktoren erfordert einen deterministischen Regelalgorithmus. Die primäre Steuerungsgröße ist das Temperatur-Delta ($\Delta T = T_{Soll} - T_{Ist}$). Um zu verhindern, dass die Systeme bei minimalen Temperaturschwankungen im Minutentakt ein- und ausschalten, was insbesondere die Kompressoren der Wärmepumpen massiv schädigen würde, muss ein Toleranzband (Hysterese) definiert werden (z. B. $\pm 0.3^\circ C$).

### Die Heiz-Strategie: Schnelle Rekuperation vs. Erhaltung

Beim Heizen muss der Algorithmus zwischen dem "Aufheizbetrieb" (großes Delta) und dem "Erhaltungsbetrieb" (kleines Delta) unterscheiden.

| Zustand          | Delta (ΔT)                                  | Systementscheidung            | Begründung (Effizienz vs. Schnelligkeit)                                                                                                                                                                                                                        |
| ---------------- | ------------------------------------------- | ----------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Kritisch zu kalt | $\Delta T > 1.5^\circ C$                    | Split-Klima AN + Erdwärme AN  | Die Fußbodenheizung ist zu träge, um den Wohnkomfort zeitnah herzustellen. Die konvektive Split-Klimaanlage wird zugeschaltet, um die Raumluft extrem schnell zu erhitzen. Parallel startet die Erdwärmepumpe, um die thermische Masse des Estrichs aufzuladen. |
| Leicht zu kalt   | $0.3^\circ C < \Delta T \le 1.5^\circ C$    | Split-Klima AUS + Erdwärme AN | Der Raum befindet sich nahe der Solltemperatur. Hier greift das Prinzip der maximalen Energieeffizienz. Die Erdwärmepumpe arbeitet mit ihrem hohen, stabilen COP und übernimmt die Erhaltungslast exklusiv.                                                     |
| Ziel erreicht    | $-0.3^\circ C \le \Delta T \le 0.3^\circ C$ | Alle Heizsysteme in Standby   | Die Raumtemperatur liegt innerhalb der Toleranzhysterese. Um Kurztakten zu vermeiden, verbleiben die Systeme im Ruhezustand.                                                                                                                                    |

Sobald im Aufheizbetrieb das Delta kleiner als $1.5^\circ C$ wird, schaltet die Split-Klimaanlage ab, und die Fußbodenheizung übernimmt den nahtlosen Übergang in den Erhaltungsbetrieb. Dies maximiert die Schnelligkeit bei der initialen Aufheizung und optimiert den Energieverbrauch, sobald die grobe Diskrepanz beseitigt ist.

### Die Kühl-Strategie: Natürliche vs. Maschinelle Kühlung

Im Kühlfall muss die natürliche Lüftung algorithmisch immer priorisiert werden, da sie keine energetischen Kosten verursacht. Hierbei stößt die Limitierung der Sensorik (nur Temperatursensoren) an ihre Grenzen, wie zuvor erläutert. Der Algorithmus prüft zunächst, ob die Außentemperatur signifikant unter der Innentemperatur liegt. Ist dies der Fall, wird dem Nutzer via Actionable Notification auf das Smartphone geraten, die Fenster zu öffnen.

Es bedarf jedoch einer Timeout-Regelung. Weilt der Nutzer nicht im Haus oder ignoriert er die Benachrichtigung, darf der Raum nicht unkontrolliert überhitzen. Der Algorithmus startet daher nach dem Versand der Benachrichtigung einen Timer (z. B. 15 Minuten). Wird das Fenster innerhalb dieses Zeitfensters nicht geöffnet (verifiziert durch Fensterkontakte), wird dies als Ablehnung der natürlichen Kühlung gewertet, und die Split-Klimaanlage übernimmt im Kühlmodus.

Ist es draußen wärmer als drinnen, entfällt der Lüftungs-Ratschlag komplett, und die maschinelle Kühlung springt direkt an. Die Fußbodenheizung (sofern sie über eine passive Kühlfunktion verfügt) wird in diesem speziellen Ablaufplan deaktiviert gelassen, da passive Bodenkühlung extrem anfällig für Kondenswasserbildung (Taupunktunterschreitung) am Boden ist, wenn keine Feuchtigkeitssensoren zur Überwachung vorhanden sind.

## Algorithmischer Ablaufplan (Pseudo-Code)

Der folgende, stark auskommentierte Pseudo-Code übersetzt die diskutierte thermodynamische Orchestrierungslogik in einen iterativen Entscheidungsbaum. Dieser Code ist dafür konzipiert, als regelmäßige Automatisierung (z.B. alle 5 Minuten) oder Event-basiert (bei Änderung von Sensorwerten oder Zieltemperaturen) durchlaufen zu werden. Er berücksichtigt strikt die Vorgabe des Nutzers, dass primär Innen- und Außentemperatur zur Verfügung stehen.

```
// =====================================================================
// PHASE 1: Initialisierung und Datenerfassung
// =====================================================================
// Auslesen der vom virtuellen Master-Thermostat geforderten Temperatur
T_target = GET_STATE('climate.virtual_master_thermostat', 'temperature')
Modus    = GET_STATE('climate.virtual_master_thermostat', 'hvac_mode')// Auslesen der verfügbaren Sensoren
T_in  = GET_STATE('sensor.indoor_temperature')
T_out = GET_STATE('sensor.outdoor_temperature')// Auslesen von Fenster-Kontakten (falls vorhanden, sonst Standardwert FALSE)
Window_Open = GET_STATE('binary_sensor.room_window') == 'on'// Definition der regelungstechnischen Konstanten
HYSTERESIS  = 0.3   // Toleranzband in Grad Celsius zur Vermeidung von Taktung
DELTA_FAST  = 1.5   // Temperaturdifferenz, ab der die Split-Klima als Booster zuschaltet
VENT_OFFSET = 1.0   // Mindest-Temperaturdifferenz nach außen, damit Lüften lohnt// Berechnung der aktuellen Regelabweichung
Delta_T = T_target - T_in// =====================================================================
// PHASE 2: Sicherheits-Interlock (Fensterüberwachung)
// =====================================================================
WENN Window_Open IST WAHR DANN
// Um massive Energieverschwendung zu vermeiden, werden alle Systeme blockiert
SETZE_STATUS('climate.erdwaermepumpe', 'off')
SETZE_STATUS('climate.split_klima', 'off')// Prüflogik: Sollte das Fenster wieder geschlossen werden?
// Hier greift die Temperaturlimitierung: Wir prüfen nur, ob der Kühleffekt
// erschöpft ist (Delta_T >= 0) oder es draußen wärmer wird als drinnen.
WENN Delta_T >= 0 ODER T_out >= T_in DANN
    WENN Letzte_Benachrichtigung('Fenster_Zu') ÄLTER_ALS 30_Minuten DANN
        SENDE_BENACHRICHTIGUNG("Zieltemperatur erreicht oder draußen wärmer. Bitte Fenster schließen, um Energie zu sparen.")
    ENDE WENN
ENDE WENN

BEENDE_LOGIK // Keine weiteren Heiz-/Kühlaktionen, solange Fenster offen
ENDE WENN// =====================================================================
// PHASE 3: Evaluierung nach globalem Betriebsmodus
// =====================================================================
WENN Modus == 'off' DANN
SETZE_STATUS('climate.erdwaermepumpe', 'off')
SETZE_STATUS('climate.split_klima', 'off')
BEENDE_LOGIK
ENDE WENN// ---------------------------------------------------------------------
// SZENARIO A: HEIZBEDARF (Raum ist zu kalt)
// ---------------------------------------------------------------------
WENN Delta_T > HYSTERESIS DANNWENN Delta_T > DELTA_FAST DANN
    // Raum ist stark ausgekühlt -> Booster-Modus für maximale Geschwindigkeit
    SETZE_STATUS('climate.split_klima', 'heat', T_target)
    SETZE_STATUS('climate.erdwaermepumpe', 'heat', T_target) // Startet parallel für thermische Masse
SONST
    // Raum ist nahe der Solltemperatur -> Maximale Effizienz (Nur Erdwärme)
    SETZE_STATUS('climate.split_klima', 'off')
    SETZE_STATUS('climate.erdwaermepumpe', 'heat', T_target)
ENDE WENN
// ---------------------------------------------------------------------
// SZENARIO B: KÜHLBEDARF (Raum ist zu warm)
// ---------------------------------------------------------------------
SONST WENN Delta_T < -HYSTERESIS DANN// Evaluierung der natürlichen Fensterlüftung (Kostenloses Kühlen)
// Bedingung: Draußen muss es signifikant kälter sein als drinnen.
WENN T_out < (T_in - VENT_OFFSET) DANN
    
    // Benachrichtigungslogik mit Spam-Schutz
    WENN Letzte_Benachrichtigung('Lueften') ÄLTER_ALS 60_Minuten DANN
        SENDE_BENACHRICHTIGUNG("Lüften empfohlen! Die Außentemperatur ist optimal, um den Raum natürlich zu kühlen.")
        STARTE_TIMER('Ventilation_Timeout', 15_Minuten)
    ENDE WENN
    
    // Prüfung, ob der Nutzer der Empfehlung gefolgt ist
    WENN Timer('Ventilation_Timeout') IST ABGELAUFEN DANN
        // Nutzer hat das Fenster ignoriert -> Fallback auf maschinelle Kühlung
        SETZE_STATUS('climate.erdwaermepumpe', 'off')
        SETZE_STATUS('climate.split_klima', 'cool', T_target)
    SONST
        // Befinden uns in der Wartezeit -> Aktive Systeme bleiben vorerst aus
        SETZE_STATUS('climate.erdwaermepumpe', 'off')
        SETZE_STATUS('climate.split_klima', 'off')
    ENDE WENN
    
SONST
    // Natürliches Lüften macht physikalisch keinen Sinn (draußen wärmer)
    // -> Direkter Start der Split-Klimaanlage zur effizienten Kühlung
    SETZE_STATUS('climate.erdwaermepumpe', 'off')
    SETZE_STATUS('climate.split_klima', 'cool', T_target)
ENDE WENN
// ---------------------------------------------------------------------
// SZENARIO C: ZIELTEMPERATUR ERREICHT (Innerhalb des Toleranzbandes)
// ---------------------------------------------------------------------
SONST
// Die Raumtemperatur entspricht dem Wunschwert (± Hysterese).
// Die schnelle konvektive Anlage wird sofort abgeschaltet.
SETZE_STATUS('climate.split_klima', 'off')// Die Fußbodenheizung kann im Modus 'Auto' verbleiben. Da ihre interne
// Regelung (idealerweise über Versatile Thermostat oder Heizkurve) 
// die Modulierung übernimmt, greifen wir hier nicht hart per 'off' ein, 
// um die Hydraulik nicht zu destabilisieren.
SETZE_STATUS('climate.erdwaermepumpe', 'auto', T_target)
ENDE WENNPraktische Implementierung in Home Assistant (YAML-Struktur)Um diesen logischen Ablaufplan in die Realität von Home Assistant zu überführen, bedarf es einer spezifischen YAML-Konfiguration, die das "Climate Template" mit den untergeordneten Automatisierungen verknüpft.Der virtuelle Master-ThermostatDie Präsentationsschicht wird in der configuration.yaml definiert. Der Parameter set_temperature ist von zentraler Bedeutung. Er schaltet keine Hardware, sondern schreibt die vom Nutzer gewählte Temperatur in einen Helfer (input_number.target_temperature) und triggert anschließend das Orchestrierungsskript.YAMLclimate:
  - platform: climate_template
    name: "Intelligente Raumsteuerung"
    unique_id: intelligent_climate_master
    modes:
      - "heat"
      - "cool"
      - "off"
    min_temp: 16
    max_temp: 28
    current_temperature_template: "{{ states('sensor.indoor_temperature') }}"
    target_temperature_template: "{{ states('input_number.target_temperature') }}"
    set_temperature:
      - action: input_number.set_value
        target:
          entity_id: input_number.target_temperature
        data:
          value: "{{ temperature }}"
      - action: script.orchestriere_klima_systeme
Das Skript script.orchestriere_klima_systeme beinhaltet die in Jinja2-Templates und Home Assistant-Bedingungen übersetzte Logik des zuvor beschriebenen Pseudo-Codes.Sub-System Management der FußbodenheizungEin kritischer Ratschlag für die praktische Implementierung betrifft die Ansteuerung der Erdwärmepumpe. Es wird davon abgeraten, die Zonenventile der Fußbodenheizung über primitive On/Off-Switches in der Automatisierung direkt zu triggern. Stattdessen sollte für die Fußbodenheizung der "Versatile Thermostat" als Sub-Regler konfiguriert werden.Die übergeordnete Orchestrierungslogik gibt dann lediglich die Zieltemperatur und den Modus (Heat/Off) an den Versatile Thermostat weiter. Dieser übernimmt mit seinem Auto-TPI-Algorithmus die feingranulare Aufgabe, das Überschwingen des Estrichs mathematisch zu antizipieren und die Ventile mit dem korrekten Tastgrad (Duty Cycle) zu takten. Dies verbindet die grobe, hybride Entscheidungsfindung auf der Meta-Ebene perfekt mit der notwendigen physikalischen Feinregelung auf der Hardware-Ebene.Benachrichtigungsmanagement für FensterlüftungDie Forderung des Nutzers nach Benachrichtigungen beim Lüften erfordert den Einsatz der notify und persistent_notification Dienste innerhalb von Home Assistant. Es ist von größter Wichtigkeit, dass das System den Nutzer nicht durch ständige Wiederholungen "spammt". Zudem muss eine Benachrichtigung zum Öffnen des Fensters logisch mit einer Benachrichtigung zum Schließen gekoppelt sein.Eine vorbildliche Automatisierung für den Schließ-Vorgang prüft, ob das Fenster geöffnet ist und ob der Kühleffekt erschöpft ist (Außentemperatur steigt über Innentemperatur).YAMLtrigger:
  - platform: numeric_state
    entity_id: sensor.outdoor_temperature
    above: sensor.indoor_temperature
condition:
  - condition: state
    entity_id: binary_sensor.room_window
    state: "on"
action:
  - action: notify.mobile_app_nutzer
    data:
      title: "Kühleffekt erschöpft"
      message: "Es ist draußen nun wärmer als drinnen. Bitte schließen Sie das Fenster."
      data:
        actions:
          - action: "FENSTER_IGNORIEREN"
            title: "Verstanden"
```

Solche "Actionable Notifications" erlauben es dem Nutzer, direkt über die Push-Nachricht auf dem Smartphone zu interagieren und den Status im System zu quittieren.

## Fazit und Handlungsempfehlungen


Die Realisierung eines bivalenten Klimatisierungssystems, das vollautomatisch zwischen der Strahlungswärme einer Erdwärmepumpe, der Konvektionskraft einer Split-Klimaanlage und der natürlichen Fensterlüftung moduliert, ist eine der anspruchsvollsten Disziplinen im Bereich des Smart Homes. Konventionelle, monolithische Integrationen scheitern hier an der notwendigen Flexibilität.

Die vorgeschlagene Lösung basiert auf einer strikten Trennung von Benutzeroberfläche und Steuerungslogik. Durch die Implementierung des "Climate Template" als Frontend-Thermostat und eines maßgeschneiderten Python/Jinja2-Skripts im Backend kann die physikalische Realität beider Heizsysteme optimal ausgenutzt werden. Die extrem hohe Energieeffizienz der Erdwärmepumpe wird zur Deckung der Grundlast herangezogen, während die herausragende Reaktionsgeschwindigkeit der Split-Klimaanlage gezielt als Booster eingesetzt wird, um Temperatur-Deltas von über $1.5^\circ C$ in Minuten zu neutralisieren.

Hinsichtlich der Kühlstrategie erfüllt die rein temperaturbasierte Auswertung (Fenster auf, wenn es draußen kälter ist) die aktuellen Anforderungen des Nutzers vollumfänglich und garantiert kostenlose Kälte. Es wird jedoch als Experten-Empfehlung nachdrücklich darauf hingewiesen, das System mittelfristig um Feuchtigkeitssensoren zu erweitern und die Integration "Thermal Comfort" zu implementieren. Nur der Vergleich der absoluten Luftfeuchtigkeit (in $g/m^3$) schützt die abgekühlten Bodenflächen langfristig und sicher vor Taupunktunterschreitungen und somit vor Schimmelbildung. Mit dieser Architektur ist das Smart Home für maximale Energieeffizienz und überlegenen thermischen Komfort optimal aufgestellt.

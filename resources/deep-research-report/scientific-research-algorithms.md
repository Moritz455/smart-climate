# Entwurf und Optimierung prädiktiver Regelungsalgorithmen für intelligente Thermostatsysteme: Integration von Raumklima, Luftqualität und Effizienzkennzahlen

## Einleitung in die softwaredefinierte Gebäudeautomation

Die Steuerung von Heizungs-, Lüftungs- und Klimaanlagen (HVAC) hat sich in den letzten Jahren von einfachen, hysterese-basierten Zweipunktreglern hin zu hochkomplexen, prädiktiven und adaptiven Systemen entwickelt. Gebäude und deren Klimatisierungssysteme sind global für mehr als fünfzig Prozent des gesamten Energieverbrauchs verantwortlich, weshalb der Optimierung von Regelungsalgorithmen eine entscheidende ökologische und ökonomische Rolle zukommt. Die herkömmliche HVAC-Steuerung stützt sich zumeist auf reaktive Prinzipien, die Temperaturabweichungen erst dann korrigieren, wenn sie bereits eingetreten sind, was zu systemischen Ineffizienzen, thermischen Schwankungen und einem erhöhten Energieverbrauch führt.

Moderne Open-Source-Plattformen im Bereich der Gebäudeautomation, wie beispielsweise die Integrationen für das Home Assistant Ökosystem, demonstrieren eindrucksvoll den Übergang zu softwaredefinierten Thermostaten. Lösungen wie der "Versatile Thermostat" oder "Better Thermostat" abstrahieren die physische Hardware (wie etwa Heizkörperthermostate, Klimaanlagen oder Fußbodenheizungsaktoren) und überlagern diese mit fortgeschrittenen mathematischen Modellen. Diese Architektur ermöglicht es, den Energieverbrauch zu minimieren und gleichzeitig den thermischen Komfort zu maximieren, indem sie maschinelles Lernen, Wetterprognosen und multidimensionale Sensordaten in Echtzeit fusionieren.

Der vorliegende Forschungsbericht widmet sich dem detaillierten Architekturentwurf eines vollumfänglichen Regelungsalgorithmus. Dieses System ist darauf ausgelegt, multidimensionale Messwerte – namentlich Raumtemperatur, absolute Luftfeuchtigkeit und Luftqualitätsindikatoren (wie CO2- oder VOC-Konzentration) – aufzunehmen und zu verarbeiten. Ein wesentlicher Innovationsgrad dieses Entwurfs liegt in der simultanen, dynamischen Berechnung der Leistungskoeffizienten (Coefficient of Performance, COP) für den Heizbetrieb sowie der Energieeffizienzquotienten (Energy Efficiency Ratio, EER) für den Kühlbetrieb. Diese Kennzahlen fließen als variable Gewichtungsfaktoren in die Kostenfunktion der prädiktiven Regelung ein. Darüber hinaus werden die theoretischen Fundamente und praktischen Implementierungen der führenden Regelungsalgorithmen, die unter anderem in fortschrittlichen Home Assistant Repositories Anwendung finden, umfassend analysiert. Dazu gehören Model Predictive Control (MPC), der proportional-integral-derivative Regler (PID) mit Auto-Tuning, Time Proportional Integral (TPI) und heuristische, zeitbasierte Künstliche Intelligenz (AI Time Based Kalibrierung). Das Endziel ist die Synthese eines Master-Algorithmus, der deterministische Umgebungsstabilisierung, thermodynamische Effizienz und adaptives maschinelles Lernen auf lokaler Hardware vereint, ohne auf Cloud-Infrastrukturen angewiesen zu sein.

## Theoretische Grundlagen der thermodynamischen Raumzustandserfassung

Ein robuster Regelungsalgorithmus erfordert zunächst eine hochpräzise Zustandsschätzung des zu regelnden Raumes. Da handelsübliche Thermostatventile (Thermostatic Radiator Valves, TRVs) bauartbedingt Temperatur- und Feuchtigkeitsmessungen in unmittelbarer Nähe des Heiz- oder Kühlelements vornehmen, sind diese Werte durch die lokale Wärmeabstrahlung stark verfälscht. Fortschrittliche Algorithmen entkoppeln daher konsequent den Sensor vom Aktor und nutzen abgesetzte Raum- und Außensensoren zur Ermittlung des wahren thermodynamischen Raumzustands.

### Die Erfassung der thermischen Behaglichkeit

Die Erfassung der Temperatur bildet die Basis der thermischen Behaglichkeitsbewertung, darf jedoch nicht isoliert betrachtet werden. Die Herausforderung besteht in der Differenzierung zwischen der bloßen Lufttemperatur und der mittleren Strahlungstemperatur (Mean Radiant Temperature, MRT), welche durch die thermische Masse von Wänden und Böden dominiert wird. Da diese Gebäudestrukturen stark verzögert auf Änderungen der Heizleistung reagieren, ist eine reine Betrachtung der Lufttemperatur unzureichend.

Zudem muss der Algorithmus die zeitliche Ableitung der Temperaturveränderung bilden, um externe Störgrößen, wie beispielsweise Fensteröffnungen, von natürlichen thermodynamischen Schwankungen zu unterscheiden. Ein rapider Temperaturabfall, der einen spezifischen Schwellenwert in Grad Celsius pro Minute überschreitet, wird als offenes Fenster identifiziert und führt zu einer sofortigen Suspendierung des Heiz- oder Kühlvorgangs. Diese softwareseitige Erkennung eliminiert die Notwendigkeit für physische Fensterkontaktsensoren in jedem Raum, wenngleich physische Sensoren für eine verzögerungsfreie Abschaltung präferiert werden.

### Transformation von relativer zu absoluter Luftfeuchtigkeit

Während herkömmliche Regelsysteme lediglich die relative Luftfeuchtigkeit (RH) in Prozent betrachten, ist dies für tiefgreifende thermodynamische Entscheidungen im HVAC-Kontext irreführend. Die relative Feuchtigkeit ist temperaturabhängig; warme Luft kann absolut mehr Wasser aufnehmen als kalte Luft. Ein Algorithmus, der Fensteröffnungen zur Schimmelprävention vorschlägt, muss daher zwingend auf Basis der absoluten Luftfeuchtigkeit agieren.

Der vorgeschlagene Algorithmus wandelt die gemessene Temperatur ($T$ in °C) und relative Feuchte ($RH$ in %) kontinuierlich in die absolute Luftfeuchtigkeit ($AH$ in g/m³) und den Taupunkt ($T_d$) um. Zur Berechnung des Sättigungsdampfdrucks ($E_s$) in Hektopascal wird die Magnus-Formel herangezogen, welche für den für Wohnräume relevanten Temperaturbereich eine exzellente Approximation liefert:

$$E_s = 6.112 \cdot \exp\left(\frac{17.67 \cdot T}{T + 243.5}\right)$$

Der tatsächliche Dampfdruck ($E$) des Wasserdampfes in der Luft ergibt sich direkt aus der relativen Luftfeuchtigkeit:

$$E = \frac{RH}{100} \cdot E_s$$

Daraus lässt sich die absolute Luftfeuchtigkeit ($AH$) berechnen, unter Verwendung der universellen Gaskonstante für Wasserdampf ($R_v \approx 461.5 \, \text{J/(kg K)}$) und der Umrechnung der Temperatur in Kelvin:

$$AH = \frac{E \cdot 100}{R_v \cdot (T + 273.15)} \cdot 1000$$

Diese Transformation, welche in fortgeschrittenen Smart-Home-Komponenten wie der "Thermal Comfort" Integration implementiert ist, ist kritisch für die Lüftungsempfehlung und Entfeuchtungssteuerung. Liegt die absolute Feuchtigkeit im Außenbereich niedriger als im Innenraum ($AH_{out} < AH_{in}$), kann der Algorithmus passive Entfeuchtung durch automatisierte Fensteröffner oder Lüftungsaufforderungen initiieren, anstatt auf eine energieintensive, aktive Kompressor-Entfeuchtung (Dry Mode) einer Klimaanlage zurückzugreifen.

Die kontinuierliche Überwachung des berechneten Taupunkts ($T_d$) verhindert zudem Kondensation und Schimmelbildung an kalten Außenwänden oder Rohrleitungen. Dies ist insbesondere bei der Verwendung von Flächenheizungen und -kühlungen essenziell, da die Vorlauftemperatur im Kühlbetrieb niemals den Taupunkt der Raumluft unterschreiten darf, um strukturelle Feuchtigkeitsschäden im Boden oder Mauerwerk zu vermeiden.

### Luftqualität, CO₂-Metriken und Enthalpie

Zusätzlich zur Temperatur und Feuchte fließen CO₂-Konzentrationen (gemessen in parts per million, ppm) und flüchtige organische Verbindungen (VOCs) in die Matrix ein. Präzise Sensoren wie der Sensirion SCD30 liefern verlässliche Daten zur Raumbelegung und Luftqualität. Hohe CO₂-Werte erfordern einen physikalischen Luftaustausch, der unweigerlich mit einem Enthalpieverlust im Winter oder einem unerwünschten Enthalpieeintrag im Sommer einhergeht.

Die Luftenthalpie ($h$) wird kontinuierlich berechnet, um den energetischen Aufwand der Konditionierung von Frischluft exakt zu quantifizieren:

$$h = c_{p,L} \cdot T + x \cdot (c_{p,D} \cdot T + \Delta h_v)$$

Dabei repräsentiert $c_{p,L}$ die spezifische Wärmekapazität der trockenen Luft, $x$ den Wassergehalt in kg Wasser pro kg trockener Luft, $c_{p,D}$ die spezifische Wärmekapazität des Wasserdampfs und $\Delta h_v$ die Verdampfungsenthalpie von Wasser. Der Entscheidungsalgorithmus wägt anhand dieser thermodynamischen Metrik permanent ab, ob der energetische Verlust durch Zwangslüftung durch den aktuellen Leistungskoeffizienten (COP/EER) des HVAC-Systems wirtschaftlich kompensiert werden kann, oder ob die Lüftung auf einen späteren Zeitpunkt mit vorteilhafteren Außentemperaturen verschoben werden sollte.

## Berechnung der Systemeffizienz: Dynamischer COP und EER

Ein Paradigmenwechsel im modernen Algorithmus-Design ist die stringente Abkehr von der reinen Temperaturverfolgung hin zu einer kosten- und effizienzoptimierten Regelung. Der Algorithmus muss zu jedem Zeitschritt evaluieren, mit welchem thermodynamischen Wirkungsgrad die angeschlossene Wärmepumpe, das Direct Expansion (DX) Klimagerät oder die Gastherme operiert.

### Der Coefficient of Performance (COP) im Heizbetrieb

Der COP definiert das Verhältnis von nutzbarer thermischer Heizleistung ($\dot{Q}_H$) zur aufgewendeten elektrischen Kompressor- und Hilfsleistung ($P_{el}$):

$$COP = \frac{\dot{Q}_H}{P_{el}}$$

Da physische Sensoren für die direkte Messung der thermischen Heizleistung auf Ebene einzelner Räume zumeist fehlen, approximiert der Algorithmus den aktuellen COP. Der theoretisch maximal erreichbare Carnot-COP ist strikt durch die Quellentemperatur (Außentemperatur, $T_{out}$) und die Senkentemperatur (Vorlauftemperatur des Heizkreises oder Raumtemperatur, $T_{in}$) in Kelvin limitiert:

$$COP_{Carnot} = \frac{T_{in} + 273.15}{T_{in} - T_{out}}$$

Reale Maschinen erreichen diesen theoretischen Wert niemals, sondern operieren mit einem spezifischen Gütegrad ($\eta_{G\ddot{u}te}$), der stark von der Teillast, der Kompressordrehzahl und der Vereisung des Außenverdampfers abhängig ist. Inverter-gesteuerte Wärmepumpen weisen eine nicht-lineare Effizienzkurve auf; sobald das Gerät moduliert und nicht unter Volllast läuft, steigt der COP im Teillastbetrieb oftmals signifikant an.

| Systemzustand                     | Einfluss auf COP    | Relevanz für den Algorithmus                                                               |
| --------------------------------- | ------------------- | ------------------------------------------------------------------------------------------ |
| Volllast (100% Inverter)          | Reduziert           | Algorithmus sollte sanfte Rampen statt abrupter Sprünge fordern.                           |
| Hohe Außentemperatur ($T_{out}$)  | Erheblich erhöht    | Algorithmus verschiebt Heizzyklen in die warme Mittagszeit.                                |
| Hohe Vorlauftemperatur ($T_{in}$) | Reduziert           | Algorithmus hält die Zieltemperatur so niedrig wie komfortabel möglich.                    |
| Abtauzyklen (Defrost)             | Drastisch reduziert | Erkennung von feuchtkalten Bedingungen zur Vermeidung von Heizlasten während des Defrosts. |

Durch die Einbindung von Wetterprognosen kann der prädiktive Algorithmus Heizzyklen in Tageszeiten verschieben, in denen $T_{out}$ ihr Tagesmaximum erreicht, um den durchschnittlichen COP drastisch zu maximieren. Die thermische Masse des Gebäudes dient in diesem Szenario als kostenloser Wärmespeicher, der während der effizienten Betriebsphasen aufgeladen wird.

### Der Energy Efficiency Ratio (EER) im Kühlbetrieb

Analog zum Heizbetrieb wird für den Kühlprozess der EER (bzw. die jahreszeitbedingte Variante SEER) berechnet. Der EER ist definiert als das Verhältnis der dem Raum entzogenen Wärmeleistung (Kühlleistung, $\dot{Q}_C$) zur hierfür benötigten elektrischen Leistung:

$$EER = \frac{\dot{Q}_C}{P_{el}}$$

Bei der Klimatisierung durch Direct Expansion (DX) Systeme tritt zusätzlich zur reinen Temperatursenkung die latente Kühllast durch Entfeuchtung auf. Fällt die Oberflächentemperatur des Innenverdampfers unter den Taupunkt der Raumluft, kondensiert das Wasser aus, wodurch der Luft massive Mengen an latenter Wärme entzogen werden. Der Algorithmus berechnet die Gesamtkühlleistung somit zwingend als Summe aus sensibler ($\dot{Q}_{sens}$) und latenter ($\dot{Q}_{lat}$) Kühlleistung. Die latente Kühlleistung ist direkt proportional zur vom System berechneten Veränderung der absoluten Feuchtigkeit im Raum.

Ein intelligenter Algorithmus erkennt zudem Zustände des "Free Cooling". Fällt die Außentemperatur unter die Innentemperatur, kann der Regelkreis den Kompressor vollständig deaktivieren und stattdessen eine mechanische Lüftung anfordern. In diesem Zustand konvergiert der effektive EER mathematisch gegen Unendlich, da lediglich die geringe elektrische Hilfsenergie für Ventilatoren anfällt, während enorme thermische Lasten abgeführt werden.

## Mathematische Modellierung der Kernalgorithmen

Um die multidimensionalen Zielgrößen Temperatur, Feuchte und Energieeffizienz miteinander in Einklang zu bringen, greifen Softwarelösungen wie "Better Thermostat" und "Versatile Thermostat" auf unterschiedliche regelungstechnische Paradigmen zurück. Die Wahl des jeweiligen Algorithmus bestimmt maßgeblich, wie das System Fehler (Abweichungen vom Sollwert) interpretiert und in Stellsignale (Öffnungsgrad von Ventilen, Anforderung von Kompressorleistung) umwandelt. Im Folgenden werden die vier primären Algorithmen detailliert analysiert.

| Algorithmus                      | Funktionsprinzip                                               | Optimaler Anwendungsfokus                                                 | Primäre Vorteile                                                | Systemische Nachteile                                     |
| -------------------------------- | -------------------------------------------------------------- | ------------------------------------------------------------------------- | --------------------------------------------------------------- | --------------------------------------------------------- |
| TPI (Time Proportional Integral) | Tastgrad-basiertes Schalten (PWM) innerhalb fester Zyklen      | Träge Heizsysteme (Fußbodenheizung), einfache Ventile ohne Positionierung | Extrem batterieschonend, reduziert Überschwingen massiv         | Schwächere Verfolgung von dynamischen Störgrößen          |
| PID mit Auto-Tuning              | Berechnung aus Proportional-, Integral- und Derivativanteil    | Räume mit stabilen Störgrößen, voll modulierende Ventile                  | Höchste Präzision, eliminiert bleibende Regelabweichungen       | Hohe Ventilaktivität, komplexe Initialeinstellung nötig   |
| MPC (Model Predictive Control)   | Optimierung über Vorhersagehorizont mithilfe eines Raummodells | Komplexe Systeme (Wärmepumpen), Berücksichtigung von Wetter und Solar     | Maximale Energieeffizienz, vorausschauende Störungseliminierung | Erheblicher Rechenaufwand, erfordert Systemidentifikation |
| AI Time Based                    | Heuristische, zeitliche Kalibrierungsanpassung der Sensoren    | Schnelle Inbetriebnahme, einfache Smart-Home-Setups ohne Systemwissen     | Zero-Config, umgeht Limitierungen "dummer" Hardware             | Weniger stabil in nicht-linearen oder komplexen Thermiken |

### Time Proportional Integral (TPI) und Pulsweitenmodulation

Der TPI-Algorithmus stellt eine signifikante Weiterentwicklung der einfachen Zweipunktregelung (On/Off oder Hysterese) dar. Anstatt ein Ventil vollständig zu öffnen und erst beim genauen Erreichen der Zieltemperatur abrupt zu schließen – was aufgrund der thermischen Trägheit des Heizkörpers oder der Fußbodenheizung unweigerlich zu massiven Überschwingern (Overshoots) führt – arbeitet TPI mit einer fest definierten Zykluszeit ($T_c$, typischerweise 10 bis 30 Minuten). Innerhalb dieses Zyklus berechnet der Algorithmus basierend auf der aktuellen Temperaturdifferenz einen Tastgrad (Duty Cycle, $D$), der spezifiziert, welchen prozentualen Anteil der Zykluszeit das Heizsystem aktiviert sein soll.

Mathematisch lässt sich der Duty Cycle durch eine vereinfachte PI-Gleichung abbilden:

$$D(t) = K_p \cdot (T_{Soll} - T_{Ist}(t)) + K_i \cdot \int_{0}^{t} (T_{Soll} - T_{Ist}(\tau)) d\tau$$

Der berechnete Duty Cycle $D$ wird anschließend durch eine PWM-Engine (Pulsweitenmodulation) in ein zeitbasiertes Schaltsignal übersetzt. Ist die Abweichung groß (z.B. nach dem morgendlichen Aufstehen), beträgt der Tastgrad 100 %. Nähert sich die Raumtemperatur dem Sollwert an, reduziert sich der proportionale Anteil, und das Ventil wird beispielsweise in einem 20-Minuten-Zyklus nur noch für 4 Minuten geöffnet und bleibt für 16 Minuten geschlossen. Diese gepulste Energiezufuhr ermöglicht es dem Heizkörper, seine Restwärme sanft an den Raum abzugeben, ohne dass die thermische Masse überhitzt. Der integrale Anteil sichert ab, dass konstante, langsame Wärmeverluste der Raumhülle (beispielsweise an kalten Wintertagen) durch einen kontinuierlichen, minimalen Dauertastgrad kompensiert werden. TPI schont zudem die Batterien von Funkthermostaten signifikant, da das Ventil nur wenige Male pro Stunde verfährt.

### Der PID Controller und passive Auto-Tuning-Verfahren

Der Proportional-Integral-Derivative (PID) Controller ist der absolute Industriestandard für kontinuierliche Regelungsprozesse. Im Gegensatz zur TPI-Methode, die in Zyklen schaltet, liefert der PID-Regler einen kontinuierlichen Stellwert $u(t)$ (beispielsweise den prozentualen Öffnungsgrad eines Ventils von 0 bis 100 %).

$$u(t) = K_p \cdot e(t) + K_i \cdot \int_{0}^{t} e(\tau) d\tau + K_d \cdot \frac{de(t)}{dt}$$

Hierbei stellt $e(t)$ die Regeldifferenz dar ($T_{Soll} - T_{Ist}$). Die drei Komponenten wirken synergetisch:

1. Proportionalanteil ($K_p$): Reagiert unmittelbar auf den aktuellen Fehler. Ein hoher $K_p$-Wert führt zu schnellem Aufheizen, birgt aber die Gefahr von Instabilität.
2. Integralanteil ($K_i$): Summiert historische Fehler über die Zeit auf. Dies ist zwingend erforderlich, um die bleibende Regelabweichung (Steady-State Error) zu eliminieren, die entsteht, da das Gebäude kontinuierlich Wärme an die kältere Umgebung verliert. Ohne Integralanteil würde die Zieltemperatur niemals exakt erreicht werden.
3. Derivativanteil ($K_d$): Dämpft das System, indem er auf die Änderungsrate des Fehlers reagiert. Er steuert prädiktiv gegen, wenn sich die Temperatur zu schnell dem Sollwert nähert, und verhindert so das Überschwingen.

Die primäre Herausforderung bei der Implementierung von PID-Reglern in der Gebäudeautomation liegt in der korrekten Parametrierung der Verstärkungsfaktoren. Da jeder Raum einzigartige thermische Eigenschaften (Isolationswerte, Fensterflächen, Heizkörperdimensionierung) aufweist, kommen in fortgeschrittenen Integrationen Auto-Tuning-Verfahren zum Einsatz. Die passive Auto-Tuning-Logik (Passive Autotuning) startet das System oftmals zunächst in einem sicheren Hysterese- oder Relais-Modus. Während dieses Modus wird die thermische Sprungantwort des Raumes auf einen standardisierten Heizimpuls beobachtet, wobei die Totzeit und die Anstiegsgeschwindigkeit der Temperaturkurve analysiert werden. Aus den Amplituden und Periodendauern der so entstandenen Schwingungen lassen sich mithilfe von Heuristiken die idealen PID-Parameter berechnen. Sobald die Parameter konvergiert sind, schaltet der Algorithmus nahtlos und für den Nutzer unsichtbar vom Hysterese-Modus in die kontinuierliche PID-Regelung um. Ein Nachteil des PID-Reglers ist die potenziell hohe Ventilaktivität, die bei unzureichend gedämpften Derivativanteilen oder starken Sensorausreißern entsteht und den mechanischen Verschleiß erhöht.

### Model Predictive Control (MPC): Die algorithmische Speerspitze

Die Modellpräduktive Regelung (MPC) repräsentiert die technische Speerspitze der hier betrachteten Algorithmen. Während PID-Regler rein reaktiv auf bereits eingetretene Messwertabweichungen reagieren, nutzt MPC ein mathematisch-physikalisches Modell des Raumes, um das zukünftige thermodynamische Verhalten über einen Vorhersagehorizont (Predictive Horizon) von mehreren Stunden zu simulieren.

Das thermische Verhalten des Gebäudes wird typischerweise über ein analoges Resistor-Capacitor (RC) Ersatzschaltbild abgebildet. Dabei repräsentieren die Wände, Fenster und die Raumluft die Wärmekapazitäten ($C$), während die Isolierung und der Wärmeübergang durch thermische Widerstände ($R$) modelliert werden. Die diskrete Zustandsraumdarstellung dieses physikalischen Modells nimmt die Form an:

$$\mathbf{x}_{k+1} = A \mathbf{x}_k + B \mathbf{u}_k + E \mathbf{d}_k$$

Hierbei ist $\mathbf{x}_k$ der Zustandsvektor (umfassend Metriken wie Raumlufttemperatur, Wandinnentemperatur, absolute Feuchtigkeit), $\mathbf{u}_k$ ist der Vektor der steuerbaren Stellgrößen (z. B. Vorlauftemperatur, Ventilöffnung, Kompressorleistung), und $\mathbf{d}_k$ repräsentiert die messbaren Störgrößen. Der herausragende Vorteil von MPC ist, dass der Verlauf dieser Störgrößen durch externe Prognosen (wie die Wettervorhersage für Außentemperatur und solare Einstrahlung sowie digitale Anwesenheitspläne) im Vorfeld bekannt ist.

Der Algorithmus löst zu jedem diskreten Zeitschritt online ein Optimierungsproblem, um eine Sequenz zukünftiger Stellbefehle zu generieren, welche eine vordefinierte Kostenfunktion ($J$) minimiert:

$$\min_{\mathbf{u}} J = \sum_{k=1}^{N} \left( \Vert{}\mathbf{x}_k - \mathbf{x}_{ref}\Vert{}_Q^2 + \Vert{}\mathbf{u}_k\Vert{}_R^2 \right)$$

In diese Kostenfunktion lassen sich direkt die zuvor erörterten Größen COP und EER integrieren. Ist beispielsweise ein starker solarer Gewinn durch Südfenster in zwei Stunden prognostiziert, simuliert der MPC-Algorithmus diesen Wärmeeintrag voraus. Er erkennt, dass ein aktuelles Aufheizen des Raumes durch das HVAC-System auf die strikte Zieltemperatur $\mathbf{x}_{ref}$ zu einer massiven Überhitzung (und damit zu thermischem Unbehagen) führen würde. Anstatt Heizenergie aufzuwenden, lässt das System die Temperatur präemptiv marginal abfallen und nutzt die anstehende Sonneneinstrahlung passiv zur Wiedererwärmung. Analog verlagert der MPC-Algorithmus den Betrieb von Luft-Wasser-Wärmepumpen bevorzugt in die Nachmittagsstunden, da hier die wärmere Außenluft den COP maximiert. Der Gebäudekörper wird als thermischer Speicher aufgeladen, um den Heizbedarf in den kalten, ineffizienten Nachtstunden zu reduzieren. Studien belegen, dass durch den Einsatz von verteiltem oder zentralisiertem MPC der Energieverbrauch von HVAC-Systemen um 15 % bis 50 % gesenkt werden kann, bei gleichzeitiger Verbesserung der Regelgüte. Der Preis für diese Effizienz ist ein drastisch erhöhter Berechnungsaufwand, der bei Edge-Geräten an Grenzen stoßen kann, sowie die absolute Notwendigkeit, durch Verfahren der Systemidentifikation (System Identification) ein akkurates RC-Gebäudemodell bereitzustellen.

### AI Time Based Kalibrierung: Die datengetriebene Heuristik

Als pragmatische und ressourcenschonende Alternative zu rechenintensiven MPC-Modellen nutzen Integrationen wie der "Better Thermostat" datengetriebene Ansätze in Form von "AI Time Based" Algorithmen. Obwohl der Begriff "Künstliche Intelligenz" in diesem Smart-Home-Kontext für heuristische, iterative Lernverfahren verwendet wird, verbirgt sich dahinter ein wirkungsmächtiger Mechanismus zur Offset-Kalibrierung von TRVs, der insbesondere Limitierungen restriktiver Hardware umgeht.

Zahlreiche kommerzielle Thermostate (beispielsweise im Zigbee- oder Homematic-Ökosystem) verwehren dem übergeordneten System den direkten Zugriff auf die physische Ventilposition. Sie erlauben lediglich die Anpassung der Zieltemperatur oder die Übertragung eines statischen Temperatur-"Offsets". Die Time-Based Logik analysiert fortlaufend die Latenzzeit zwischen einer algorithmischen Offset-Anpassung und der daraufhin tatsächlich messbaren Temperaturveränderung am externen Referenzsensor in der Raummitte. Das System extrahiert aus empirischen Heizzyklen, wie stark das Thermostat künstlich "gepusht" werden muss (sogenannter Aggressive Mode), um träge Räume in einer tolerablen Zeitspanne $t$ auf Zieltemperatur zu bringen.

Erreicht die Temperatur zu schnell den Zielbereich und schwingt über, dämpft der Lernalgorithmus zukünftige Offset-Verschiebungen. Dieser Ansatz besticht durch seine Einfachheit: Er erfordert im Gegensatz zu MPC keinerlei physikalische Gebäudeparameter, kein komplexes Auto-Tuning wie beim PID-Regler und ist hochgradig robust gegenüber stochastischen Änderungen der Raumnutzung. Er erreicht jedoch in hochdynamischen Szenarien, wie plötzlichen Wetterumschwüngen oder simultanen Kühlanforderungen, systembedingt nicht die mathematische Präzision eines modellbasierten Ansatzes.

## Synthese des Master-Algorithmus: Architektur und multikriterielle Logik

Basierend auf der tiefgehenden Analyse der Sensordatenerfassung, der thermodynamischen Effizienzgrößen und der bestehenden Kontrolltheorien wird im Folgenden ein neuartiger, hybrider Master-Algorithmus spezifiziert. Dieser Entwurf fusioniert Temperatur, absolute Feuchtigkeit, CO₂-Werte und Wetterprognosen und trifft multikriterielle Entscheidungen unter kontinuierlicher Berücksichtigung von COP und EER.

### Schichtenarchitektur des Regelkreises

Die Architektur des konzipierten Systems gliedert sich in drei zentrale Schichten (Layers), um Abstraktion, Skalierbarkeit und Fehlerresilienz zu garantieren:

1. State Estimation Layer: Diese Datenfusionsschicht konsolidiert rohe Sensorik (Temperatur, RH, CO₂) und transformiert sie in physikalisch nutzbare thermodynamische Vektoren (Absolute Feuchte, Taupunkt, Enthalpie). Sie ist zuständig für die Signalglättung und Filterung von Sensorausreißern.
2. Prediction & Optimization Layer: Nutzt ein vereinfachtes MPC-Modell, das fortlaufend durch historische Datensätze trainiert wird (System Identification). Dieser Layer antizipiert prädiktive Störungen (Wetter, Sonnenschein, kalenderbasierte Anwesenheitspläne) und optimiert die Betriebszeitpunkte basierend auf den berechneten Leistungskoeffizienten.
3. Actuation & Low-Level Control Layer: Übersetzt die vom MPC geforderten Wärmeströme ($\dot{Q}_{Soll}$) je nach angeschlossenem Endgerät in systemspezifische Hardwaresignale. Für "dumme" On/Off-Relais oder Boiler wird die PWM-Modulation via TPI angewandt. Für voll modulierende Smart-Ventile kommt der untergeordnete PID-Regler zum Einsatz, und für stark limitierte TRVs, die nur Temperatur-Offsets akzeptieren, agiert die AI Time Based Heuristik zur Manipulation des Thermostat-Sensors.

### Die Erweiterte Multikriterielle Kostenfunktion

Die komplexe Entscheidungsfindung des Thermostat-Algorithmus reduziert sich mathematisch auf die Minimierung einer multikriteriellen Kostenfunktion $J$ über den Prädiktionshorizont. Diese Funktion wägt thermische Behaglichkeit, Luftqualität, Struktur- und Schimmelprävention sowie den monetären und ökologischen Energieaufwand dynamisch gegeneinander ab:

$$J = \sum_{k=t}^{t+N} \left[ \alpha (T_k - T_{ref})^2 + \beta \cdot f(AH_k, T_d) + \gamma \cdot g(CO2_k) + \delta \left( \frac{P_{el,k}(T_k, T_{out,k})}{COP_k} \right) \right]$$

- Thermische Behaglichkeit ($\alpha$): Pönalisiert quadratisch jede Abweichung von der vom Nutzer eingestellten Präferenztemperatur ($T_{ref}$). Wenn Anwesenheitssensoren (Geo-Fencing oder PIR-Motion) eine Abwesenheit melden (Away-Mode), erweitert der Algorithmus dynamisch das Toleranzband (Deadband), sodass marginale Temperaturschwankungen nicht sofort zu einer kostenintensiven Aktivierung des HVAC-Systems führen. Die Integration von Presets (Comfort, Eco, Sleep) modifiziert $T_{ref}$ ohne händische Nutzereingriffe.
- Feuchtigkeits- und Schimmelprävention ($\beta$): Die nicht-lineare Straffunktion $f$ überwacht die absolute Feuchte und den Abstand der Raumtemperatur zum Taupunkt. Nähert sich die simulierte Wandtemperatur gefährlich dem Taupunkt an, steigt dieser Kostenfaktor exponentiell. Das System erzwingt dann eine Reaktion: Entweder die Raumtemperatur wird leicht angehoben (um die relative Feuchte an den Wänden zu senken), oder, falls $AH_{außen} < AH_{innen}$, generiert das System über Push-Benachrichtigungen eine Aufforderung an den Nutzer zur Zwangslüftung.
- Luftqualität ($\gamma$): Überschreitet der CO₂-Wert oder die VOC-Konzentration vordefinierte Grenzwerte (beispielsweise > 1000 ppm), wird die Notwendigkeit der Frischluftzufuhr priorisiert. Bei Vorhandensein einer integrierten mechanischen Lüftungsanlage (VMC) mit Wärmerückgewinnung kann der Algorithmus den Volumenstrom autark erhöhen, andernfalls wird auch hier der Nutzer informiert.
- Energetische Kosten und COP-Faktor ($\delta$): Dies ist der entscheidende Effizienzhebel. Die erwartete aufzuwendende elektrische Energie wird durch den prognostizierten COP der Anlage dividiert. Erwartet der Vorhersage-Layer für die anstehende Nacht Frost (-2°C), erkennt das System, dass der COP extrem einbrechen wird. Der Term $\delta$ zwingt den Algorithmus dazu, das Gebäude nachmittags präventiv bei hohem COP leicht über $T_{ref}$ aufzuheizen, um den ineffizienten Nachtbetrieb der Wärmepumpe obsolet zu machen.

### Hierarchische Entscheidungslogik bei widersprüchlichen Zuständen

Der Algorithmus verarbeitet regelmäßig Zustände, in denen Temperatur-, Feuchtigkeits- und Effizienzanforderungen diametral im Konflikt stehen. Ein typisches Szenario in der Übergangszeit oder im Sommer lautet: Der Raum ist zu warm (hoher Kühlbedarf), gleichzeitig ist die Raumluft zu feucht ($AH_{innen} > AH_{außen}$), während eine unkoordinierte Fensteröffnung einen massiven Effizienzverlust durch das Entweichen zuvor konditionierter Luft zur Folge hätte.

Um Oszillationen in der Regelung zu vermeiden, implementiert der Algorithmus eine strikt hierarchische State-Machine:

1. Level 1: Hardware-Locks und Sicherheit: Um Verschleiß an Relais oder Kältekompressoren durch Short-Cycling (schnelles Takten) zu verhindern, evaluiert der Actuation Layer zuerst minimale Einschalt- und Ausschaltzeiten ($t_{min\_on}$, $t_{min\_off}$). Diese Hard Limits überschreiben bedingungslos alle rechnerischen Outputs der PID- oder TPI-Gleichungen. Weiterhin öffnet eine Antiblockierfunktion (Summer Anti-seize) die mechanischen Ventile periodisch, um Verkalkung vorzubeugen.
2. Level 2: Fenster-Erkennung und Suspendierung: Fällt die gemessene Temperatur rapider als ein berechneter Gradient (z.B. > 0.5 °C/min), wird zwingend der Zustand "Fenster offen" getriggert. Heizung und Kompressorkühlung werden sofort blockiert, da die Energieverschwendung ansonsten maximiert würde.
3. Level 3: Enthalpie-Vergleich und Lüftungsstrategie: Ist das Fenster geschlossen, vergleicht der Algorithmus die absolute Luftfeuchtigkeit innen und außen. Ist die Außenluft trotz eventuell höherer Temperatur absolut trockener, wird passives Entfeuchten durch Fensteröffnung priorisiert, um den energieintensiven Kompressorbetrieb (EER-Degradation) zu vermeiden.
4. Level 4: Aktive Entfeuchtung vs. Sensible Kühlung: Wird die Lüftungsempfehlung nicht umgesetzt oder ist die Außenluft feuchtwarm, berechnet das MPC-Modell, ob der Kühlmodus ausreicht, um den Raum unter den Taupunkt zu bringen und die Feuchtigkeit auszukondensieren. Die anfallende latente Last wird zur Vorhersage der benötigten elektrischen Leistung in die EER-Berechnung integriert.
5. Level 5: Anwesenheitskopplung: Meldet das Geofencing die Abwesenheit aller Bewohner, werden die Faktoren für Temperaturkomfort ($\alpha$) und Luftqualität ($\gamma$) signifikant reduziert. Die Schimmelprävention ($\beta$) bleibt jedoch zwingend mit hoher Priorität aktiv, da strukturelle Feuchtigkeitsschäden an der Bausubstanz unabhängig von der menschlichen Anwesenheit vermieden werden müssen.

### Abstraktion von Hardwareschnittstellen und Fehlerresilienz (Fallback)

Ein oft vernachlässigter Aspekt beim Entwurf von Regelungsalgorithmen ist die Fehlerresilienz in hochgradig fragmentierten IoT-Netzwerken. Verlieren externe Zigbee-Temperatursensoren das Funksignal, darf das System den Aktor nicht in einem permanent offenen Zustand (Dauerheizen) belassen.

Die Architektur sieht vor, dass bei einem Signalverlust die letzte bekannte Temperatur über einen Exponentiell Geglätteten Gleitenden Durchschnitt (Exponential Moving Average, EMA) temporär für eine vordefinierte Zeitspanne fortgeführt wird. Bleibt der Sensorausfall bestehen, degradiert das System deterministisch (Graceful Degradation): Der High-Level MPC- oder PID-Controller wird deaktiviert, und der Algorithmus übergibt die Kontrolle zurück an die primitive interne Regelung des Heizkörperthermostats, oder er wechselt in einen sicheren Anti-Freeze-Modus, bis die Netzwerk-Anomalie behoben ist.

## Schlussbetrachtung und Implikationen

Der Entwurf dieses detaillierten Thermostat-Algorithmus demonstriert den technologischen Paradigmenwechsel von statischer, einkanaliger Regelungstechnik hin zu einer holistischen Raumklimasystematik. Durch die Fusion von hochauflösender Sensorik für Temperatur, absolute Luftfeuchtigkeit und Luftqualität lässt sich die Gebäudeklimatisierung exakt auf die physikalischen Notwendigkeiten der Bausubstanz und die physiologischen Bedürfnisse der Nutzer abstimmen.

Die prädiktive Integration der Effizienzkennzahlen COP und EER direkt in das Herz des Algorithmus stellt einen entscheidenden, skalierbaren Hebel zur Dekarbonisierung des globalen Gebäudesektors dar. Systeme operieren in dieser Architektur nicht länger als isolierte Inseln, sondern im ständigen thermodynamischen Abgleich mit den externen Umgebungsbedingungen und der thermischen Trägheit des Bauwerks.

Die Gegenüberstellung der Kontrollalgorithmen aus Open-Source-Plattformen verdeutlicht zudem unmissverständlich, dass keine universelle Kontrolllösung für alle Szenarien existiert. Während TPI und heuristische AI-Ansätze in stark fragmentierten, simplen Smart-Home-Umgebungen durch extreme Robustheit, Schonung der Hardware und leichte Konfigurierbarkeit brillieren, entfaltet das Model Predictive Control (MPC) – unterstützt durch präzise RC-Gebäudemodelle – das unbestreitbare Maximum an Energieeinsparpotenzial. Durch die intelligente Kaskadierung dieser Logiken – MPC für die vorausschauende High-Level-Strategie, kombiniert mit PID oder TPI für die präzise, hardwarenahe Aktorik – entsteht ein resilientes, zukunftssicheres System. Es antizipiert Störungen, verhindert Feuchtigkeitsschäden proaktiv, kommuniziert mit dem Nutzer bei Bedarf und steuert komplexe Wärmepumpen und Klimageräte stets an ihrem thermodynamischen Optimum. Die automatisierte Gebäudesteuerung wandelt sich somit fundamental von einem passiven, reaktiven Komfort-Feature zu einem aktiven, integralen Bestandteil des intelligenten Energiemanagements.

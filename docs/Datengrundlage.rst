~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Datengrundlage und Standardparameter
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Das OWP-Tool stellt historische Zeitreihen sowie technische, wirtschaftliche und optimierungsspezifische Standardparameter zur Verfügung. Ein Teil dieser Datengrundlage geht auf das Forschungsprojekt Hochtemperaturwärmepumpen in der Nah- und Fernwärmeversorgung (HTWP) zurück [HTWP2024]_. Die nachfolgenden Angaben dokumentieren die Herkunft der mitgelieferten Daten und die Bedeutung der voreingestellten Werte.

Die Standardwerte dienen als Ausgangspunkt für die Konfiguration eines Energiesystems. Sie sind weder als allgemeingültige Auslegungswerte noch als aktuelle Marktpreise oder rechtsverbindliche Angaben zu verstehen. Insbesondere Investitionskosten, Wirkungsgrade, Preisbestandteile und Förderannahmen sind für den jeweiligen Anwendungsfall zu überprüfen und gegebenenfalls anzupassen. Die angegebenen Werte beziehen sich auf den Ausgangszustand der Parametrierung; durch Auswahl von Datensätzen, Skalierung, Benutzereingaben oder den Import eines Energiesystems können sie überschrieben werden.

**Hinweis zum Bearbeitungsstand:** Leere Quellenfelder und mit **TODO** gekennzeichnete Erläuterungen markieren Daten oder Annahmen, deren Herkunft bzw. Übertragbarkeit noch zu klären ist. Interne Bedien- und Modellvorgaben sind als solche gekennzeichnet und benötigen nicht zwingend eine externe Literaturquelle. Eine Übereinstimmung mit dem HTWP-Bericht belegt die dort dokumentierte Herkunft des Wertes, nicht automatisch seine Eignung für jede heutige Anwendung.

Übersicht der Eingabedateien
============================

Die Daten werden aus dem Verzeichnis ``src/owp_milp_optimization/input/`` geladen. Die Seite ``pages/00_Energiesystem.py`` enthält weitere unmittelbar im Programm definierte Startwerte. Die Modellverwendung der Parameter ist in ``model.py`` implementiert.

.. list-table::
   :header-rows: 1
   :widths: 36 64

   * - Datei
     - Inhalt
   * - ``heat_load.csv``
     - Historische Wärmelastzeitreihen
   * - ``eco_data.csv``
     - Strom- und Gaspreise, CO₂-Zertifikatspreise, Strommix-Emissionsfaktoren und solare Einstrahlung
   * - ``param_units.json``
     - Anlagenspezifische Standardparameter
   * - ``param_opt.json``
     - Wärmenetzkosten, wirtschaftliche und Optimierungsparameter
   * - ``param_bound.json``
     - Historische Strompreisbestandteile für Netzbezug und Eigennutzung

Wärmeversorgungsdaten
=====================

Die mitgelieferten Wärmelastprofile beruhen auf Messdaten realer Fernwärmenetze. Sie können für ein ausgewähltes Jahr, einen kürzeren Zeitraum oder in skalierter Form verwendet werden. Alternativ lassen sich eigene Lastdaten importieren.

.. list-table::
   :header-rows: 1
   :widths: 29 20 19 32

   * - Parameter / Datensatz
     - Standardwert / Zeitraum
     - Quelle
     - Erläuterung
   * - Flensburg
     - 2014–2016
     - [FL2016]_
     - Stündliche Wärmelast des Fernwärmenetzes.
   * - Flensburg
     - 2017–2019
     - [FL2019]_
     - Stündliche Wärmelast des Fernwärmenetzes.
   * - Flensburg
     - 2020–2024
     - [FL2024]_
     - Stündliche Wärmelast des Fernwärmenetzes.
   * - Sønderborg
     - 2016–2019
     - [SO2019]_
     - Messdaten des Fernwärmenetzes.
   * - Referenzhaushalte Flensburg
     - 50.000
     -
     - **TODO** – Bezugsgröße für die proportionale Skalierung nach Haushaltszahl.
   * - Referenzhaushalte Sønderborg
     - 13.000
     -
     - **TODO** – Bezugsgröße für die proportionale Skalierung nach Haushaltszahl.

Im HTWP-Projekt wurden Datenlücken im Sommer 2019 für Flensburg mittels einer kubischen Regression der Wärmelast gegenüber der Außentemperatur geschlossen. Die Temperaturmesswerte stammen aus dem Climate Data Center des DWD [DWDTemp]_; als methodischer Hintergrund dient [Hellwig2003]_. Die Viertelstundenwerte aus Sønderborg wurden durch Mittelwertbildung auf Stundenwerte aggregiert. Details zur damaligen Aufbereitung sind im Anhang des HTWP-Berichts beschrieben [HTWP2024]_. Die exakte Vorverarbeitung der im OWP-Tool enthaltenen CSV-Datei, insbesondere der später ergänzten Daten, ist noch nicht vollständig dokumentiert.

Energiepreise und Emissionsfaktoren
===================================

Die historischen Zeitreihen werden aus ``eco_data.csv`` gelesen. Für Strom- und Gaspreise, Strommix-Emissionsfaktoren und CO₂-Zertifikatspreise stehen alternativ konstante Werte oder eigene Datensätze zur Verfügung. Die Preise und Emissionsfaktoren sind keine Prognosen.

.. list-table::
   :header-rows: 1
   :widths: 29 20 19 32

   * - Parameter / Datensatz
     - Standardwert / Zeitraum
     - Quelle
     - Erläuterung
   * - Stromspotmarktpreis (``el_spot_price``)
     - €/MWh
     - [Agora]_
     - Historische Day-Ahead-Preise der Strombörse, stündlich.
   * - Erdgaspreis (``gas_price``)
     - €/MWh
     - [EEX2021]_
     - Historische Preise auf Basis des EGIX-Erdgaspreisindex der PEGAS.
   * - CO₂-Zertifikatspreis (``co2_price``)
     - €/t CO₂
     -
     - **TODO** – Historische Preiszeitreihe; Datenanbieter und Zertifikatstyp ergänzen.
   * - Emissionsfaktor des Strommix (``ef_om``)
     - kg CO₂/MWh
     - [HTWP2024]_; [AgoraDoku]_
     - Eigene Berechnung des zeitabhängigen Gesamtmix-Emissionsfaktors, siehe unten.
   * - Konstanter Strompreis
     - 80 €/MWh
     -
     - **TODO** – Alternativer Startwert bei Auswahl „Konstant“.
   * - Konstanter Erdgaspreis
     - 65 €/MWh
     -
     - **TODO** – Alternativer Startwert bei Auswahl „Konstant“.
   * - Konstanter CO₂-Zertifikatspreis
     - 30 €/t CO₂
     -
     - **TODO** – Alternativer Startwert bei Auswahl „Konstant“.
   * - Konstanter Strommix-Emissionsfaktor
     - 350 kg CO₂/MWh
     -
     - **TODO** – Alternativer Startwert bei Auswahl „Konstant“.
   * - Emissionsfaktor Erdgas (``ef_gas``)
     - 0,2012 t CO₂/MWh
     - [Konstantin2013]_; [HTWP2024]_
     - Konstanter brennstoffbezogener Emissionsfaktor; für andere Brennstoffe anzupassen.

Berechnung der Strommix-Emissionsfaktoren
-----------------------------------------

Die Emissionsfaktoren des deutschen Gesamtstrommix werden auf Grundlage der im HTWP-Bericht beschriebenen Agorameter-Methodik berechnet [HTWP2024]_, [AgoraDoku]_. Die zeitaufgelösten Nettostromerzeugungsdaten nach Energieträgern werden mit Jahresdaten der AG Energiebilanzen abgeglichen. Mithilfe von Annahmen zum Kraftwerkseigenbedarf, den Nettowirkungsgraden und den brennstoffspezifischen Emissionsfaktoren werden die Emissionen je Anlagentyp berechnet und über die Stromerzeugung zum Gesamtmix zusammengeführt.

.. list-table::
   :header-rows: 1
   :widths: 30 70

   * - Datengrundlage
     - Verwendung
   * - [ENTSOE]_
     - Zeitaufgelöste Stromerzeugung nach Energieträgern
   * - [AGEB]_
     - Jahreswerte der Bruttostromerzeugung und Korrektur der Erzeugungsdaten
   * - [AgoraDoku]_
     - Methodik einschließlich Annahmen zum Kraftwerkseigenbedarf (Öko-Institut), Wirkungsgraden und Emissionsfaktoren
   * - [Juhrich2016]_
     - Brennstoffspezifische CO₂-Emissionsfaktoren
   * - [UBA2023]_
     - Vergleich und Validierung der Berechnung im HTWP-Projekt

Für 2019 ergab die HTWP-Berechnung einen mittleren Emissionsfaktor von rund 412 kg CO₂/MWh gegenüber 411 kg CO₂/MWh laut Umweltbundesamt. Die im Bericht zusätzlich untersuchte Verdrängungsmix-Methodik ist von der hier verwendeten Gesamtmix-Bilanzierung zu unterscheiden.

Solare Einstrahlung
===================

Die Einstrahlungszeitreihen werden für drei Standorte bereitgestellt. Die Anzeige erfolgt in Wh/m²; intern liegen die Werte entsprechend der Umrechnung im Quellcode in MWh/m² vor. Die Wärmeproduktion wird im Modell durch Multiplikation mit Kollektorfläche und konstantem Kollektorwirkungsgrad ermittelt.

.. list-table::
   :header-rows: 1
   :widths: 29 20 19 32

   * - Parameter / Datensatz
     - Standardwert / Zeitraum
     - Quelle
     - Erläuterung
   * - Schleswig (``solar_heat_flow_schleswig``)
     - Historische Zeitreihe
     -
     - **TODO** – DWD-Strahlungsdaten als belegte Grundlage des HTWP-Standorts [DWDStrahlung]_, [HTWP2024]_; exakte Herkunft, Aufbereitung der heutigen OWP-Reihe und Bezugsfläche klären.
   * - Chemnitz (``solar_heat_flow_chemnitz``)
     - Historische Zeitreihe
     -
     - **TODO** – Datenquelle, Messstation und Aufbereitung ergänzen.
   * - Stuttgart (``solar_heat_flow_stuttgart``)
     - Historische Zeitreihe
     -
     - **TODO** – Datenquelle, Messstation und Aufbereitung ergänzen.

Im HTWP-Projekt wurden Global- und Diffusstrahlung auf eine geneigte Kollektorebene umgerechnet (DIN 5034-2 [DIN5034]_). Die thermische Nutzbarkeit wurde damals zusätzlich anhand temperaturabhängiger Kollektorkennwerte nach [Quaschning2015]_ und [SolarKeymark2018]_ berechnet. Diese alte Vorkalkulation darf nicht ungeprüft mit der aktuellen, vereinfachten OWP-Modellierung gleichgesetzt werden. **TODO:** Für alle drei OWP-Standorte die Einstrahlungsgröße (horizontal oder geneigte Ebene), Neigung, Ausrichtung, gegebenenfalls Albedo und Vorverarbeitung dokumentieren; insbesondere eine doppelte Anwendung des Kollektorwirkungsgrades ausschließen.

Technische Standardparameter der Versorgungsanlagen
===================================================

Die Anlagenparameter sind in ``param_units.json`` hinterlegt. Nennleistungen und Leistungsgrenzen beziehen sich bei den wärmeerzeugenden Anlagen auf die thermische Ausgangsleistung; bei Solarthermie auf die Kollektorfläche, bei Speichern auf die speicherbare Wärmemenge. Wird die Kapazitätsoptimierung aktiviert, sind die vorgegebenen Minimal- und Maximalkapazitäten anstelle der festen Nennkapazität relevant.

Anlagenübergreifende Parameter
------------------------------

.. list-table::
   :header-rows: 1
   :widths: 29 20 19 32

   * - Parameter / Datensatz
     - Standardwert / Zeitraum
     - Quelle
     - Erläuterung
   * - Kapazität optimieren (``invest_mode``)
     - Nein (alle Anlagen)
     - Interne Modell-/Bedienvorgabe
     - Standardmäßig Einsatzoptimierung mit vorgegebener Kapazität.
   * - Nennwärmeleistung (``cap_N``)
     - 353 MW (WP, GuD, BHKW, Gaskessel, Elektrodenheizkessel)
     -
     - **TODO** – Wert entspricht gerundet der Engpassleistung des HTWP-Flensburger Bestandsnetzes; Übertragung auf alle Nennleistungen nicht belegt.
   * - Maximale Wärmekapazität (``cap_max``)
     - 353 MW (WP, GuD, BHKW, Gaskessel, Elektrodenheizkessel, externe Wärmequelle)
     -
     - **TODO** – Historischer Bezug zur Flensburger Engpassleistung plausibel; Übertragung auf alle Anlagentypen klären.
   * - Minimale Kapazität (``cap_min``, ``A_min``, ``Q_min``)
     - 0 MW / m² / MWh
     - Interne Modell-/Bedienvorgabe
     - Erlaubt auch die Nichtinvestition in eine Anlage.
   * - Relative maximale Last (``Q_rel_max``)
     - 100 % (WP, GuD, BHKW, Gaskessel, Elektrodenheizkessel)
     - Interne Modell-/Bedienvorgabe
     - Normierte obere Betriebsgrenze bezogen auf die Nennwärmeleistung.

Wärmepumpe (WP)
---------------

.. list-table::
   :header-rows: 1
   :widths: 29 20 19 32

   * - Parameter / Datensatz
     - Standardwert / Zeitraum
     - Quelle
     - Erläuterung
   * - Leistungszahl (``cop``)
     - 3,0
     -
     - **TODO** – Konstanter COP; im HTWP-Projekt wurde die Leistungszahl temperaturabhängig ermittelt.
   * - Relative Mindestlast (``Q_rel_min``)
     - 30 %
     -
     - **TODO** – HTWP verwendet 30 % Mindestteilmassenstrom; die Übertragbarkeit auf die thermische Mindestlast ist zu klären.

Gas- und Dampfkraftwerk (GuD)
-----------------------------

.. list-table::
   :header-rows: 1
   :widths: 29 20 19 32

   * - Parameter / Datensatz
     - Standardwert / Zeitraum
     - Quelle
     - Erläuterung
   * - Thermischer Wirkungsgrad (``eta_th``)
     - 50 %
     -
     - **TODO** – Konstanter Wert; abweichend von HTWP.
   * - Elektrischer Wirkungsgrad (``eta_el``)
     - 35 %
     -
     - **TODO** – Konstanter Wert; abweichend von HTWP.
   * - Relative Mindestlast (``Q_rel_min``)
     - 15 %
     -
     - **TODO** – Abweichend von der HTWP-Mindestlast von 30 %.

Blockheizkraftwerk (BHKW)
-------------------------

.. list-table::
   :header-rows: 1
   :widths: 29 20 19 32

   * - Parameter / Datensatz
     - Standardwert / Zeitraum
     - Quelle
     - Erläuterung
   * - Thermischer Wirkungsgrad (``eta_th``)
     - 45 %
     -
     - **TODO** – Konstanter Wert; abweichend von HTWP.
   * - Elektrischer Wirkungsgrad (``eta_el``)
     - 30 %
     -
     - **TODO** – Konstanter Wert; abweichend von HTWP.
   * - Relative Mindestlast (``Q_rel_min``)
     - 15 %
     -
     - **TODO** – Herkunft des vereinfachten Standardwerts ergänzen.

Gaskessel
---------

.. list-table::
   :header-rows: 1
   :widths: 29 20 19 32

   * - Parameter / Datensatz
     - Standardwert / Zeitraum
     - Quelle
     - Erläuterung
   * - Thermischer Wirkungsgrad (``eta``)
     - 95 %
     - [HTWP2024]_
     - Anhang „Modellierung der Versorgungsanlagen“; im Fließtext des Berichts wird abweichend 98 % genannt.
   * - Relative Mindestlast (``Q_rel_min``)
     - 15 %
     - [HTWP2024]_
     - Entspricht dem Wert im HTWP-Anhang.

Elektrodenheizkessel
--------------------

.. list-table::
   :header-rows: 1
   :widths: 29 20 19 32

   * - Parameter / Datensatz
     - Standardwert / Zeitraum
     - Quelle
     - Erläuterung
   * - Thermischer Wirkungsgrad (``eta``)
     - 95 %
     -
     - **TODO** – Konstanter Wirkungsgrad.
   * - Relative Mindestlast (``Q_rel_min``)
     - 15 %
     -
     - **TODO** – Relative untere Betriebsgrenze.

Solarthermie
------------

.. list-table::
   :header-rows: 1
   :widths: 29 20 19 32

   * - Parameter / Datensatz
     - Standardwert / Zeitraum
     - Quelle
     - Erläuterung
   * - Nennkollektorfläche (``A_N``)
     - 50.000 m²
     -
     - **TODO** – Feste Kollektorfläche für Einsatzoptimierung.
   * - Maximale Kollektorfläche (``A_max``)
     - 150.000 m²
     -
     - **TODO** – Obergrenze der Flächenoptimierung.
   * - Kollektorwirkungsgrad (``eta_col``)
     - 30 %
     -
     - **TODO** – Konstanter Wirkungsgrad; nicht mit den temperaturabhängigen HTWP-Kennwerten gleichzusetzen.

Wärmespeicher
-------------

.. list-table::
   :header-rows: 1
   :widths: 29 20 19 32

   * - Parameter / Datensatz
     - Standardwert / Zeitraum
     - Quelle
     - Erläuterung
   * - Nennspeicherkapazität (``Q_N``)
     - 5.000 MWh
     -
     - **TODO** – Feste Speicherkapazität.
   * - Maximale Speicherkapazität (``Q_max``)
     - 8.472 MWh
     - [HTWP2024]_
     - Entspricht der Obergrenze des HTWP-Kurzzeitspeichers (353 MW × 24 h).
   * - Verhältnis Ladeleistung zu Kapazität (``Q_in_to_cap``)
     - 0,041666667 h⁻¹
     -
     - **TODO** – Entspricht einer vollständigen Ladung in 24 h bei Nennleistung; Herkunft der Vorgabe klären.
   * - Verhältnis Entladeleistung zu Kapazität (``Q_out_to_cap``)
     - 0,041666667 h⁻¹
     -
     - **TODO** – Entspricht einer vollständigen Entladung in 24 h bei Nennleistung; Herkunft der Vorgabe klären.
   * - Relative Wärmeverluste (``Q_rel_loss``)
     - 0,005 % pro h
     - [HTWP2024]_; [DEA2018]_
     - Im HTWP-Projekt anhand gemessener jährlicher Verluste realer Speicher iterativ kalibriert; Verluste proportional zum Speicherinhalt.
   * - Initialspeicherstand (``init_storage``)
     - 50 %
     - [HTWP2024]_
     - Entspricht der Vorgabe für den saisonalen Speicher im HTWP-Anhang; der Kurzzeitspeicher wurde dort mit 0 % initialisiert.
   * - Ausgeglichener Speicherstand (``balanced``)
     - Ja
     - [HTWP2024]_
     - Speicherinhalt am Ende des Betrachtungszeitraums entspricht dem Anfangswert.

Externe Wärmequelle
-------------------

.. list-table::
   :header-rows: 1
   :widths: 29 20 19 32

   * - Parameter / Datensatz
     - Standardwert / Zeitraum
     - Quelle
     - Erläuterung
   * - Nennwärmeleistung (``cap_N``)
     - 10 MW
     -
     - **TODO** – Feste Leistung der externen Wärmequelle.
   * - Konstante Wärmeeinspeisung (``fix``)
     - Nein
     - Interne Modell-/Bedienvorgabe
     - Bei Aktivierung wird die volle vorgegebene bzw. optimierte Leistung in jedem Zeitschritt eingespeist; sonst bedarfsabhängiger Einsatz.

Investitions- und Betriebskosten der Versorgungsanlagen
=======================================================

Die Kostenansätze aus ``param_units.json`` werden linear auf die jeweilige Bezugsgröße bezogen. Die nachfolgenden Angaben unterscheiden Investitionskosten, fixe jährliche Betriebskosten und variable Betriebskosten. Die Technologiedatenkataloge von Energinet und der Danish Energy Agency [DEA2020]_, [DEA2018]_ sind wichtige Grundlagen des HTWP-Berichts. Das Bezugsjahr historischer Kostenwerte ist bei einer Übertragung auf heutige Projekte zu berücksichtigen.

**Hinweis zu KWK-Anlagen:** Der HTWP-Bericht bezieht die Investitionskosten und fixen Betriebskosten von GuD und BHKW auf die elektrische Nennleistung. Im aktuellen OWP-Modell wird die thermische Nennleistung für den Investitionsfluss verwendet, während bei der nachgelagerten Kostenberechnung die elektrische Nennleistung aus den Wirkungsgraden ermittelt wird. Daher sind die numerisch identischen Kostenansätze als Literaturwerte nachvollziehbar, aber ihre konsistente Bezugsgröße in Optimierung und Auswertung ist noch zu prüfen. Die variablen KWK-Betriebskosten beziehen sich auf die elektrische Arbeit.

.. list-table::
   :header-rows: 1
   :widths: 29 20 19 32

   * - Parameter / Datensatz
     - Standardwert / Zeitraum
     - Quelle
     - Erläuterung
   * - WP – Investitionskosten
     - 510.000 €/MW
     -
     - **TODO** – HTWP enthält komponentenbasierte Kostenkalkulationen, jedoch keine belegte Herleitung dieses Pauschalwertes.
   * - WP – fixe Betriebskosten
     - 4.000 €/MW/a
     - [DEA2020]_
     - Bezugsgröße thermische Nennleistung.
   * - WP – variable Betriebskosten
     - 1,20 €/MWh
     - [DEA2020]_
     - Bezugsgröße erzeugte Wärme.
   * - GuD – Investitionskosten
     - 880.000 €/MW
     - [DEA2020]_
     - HTWP: elektrische Nennleistung; TODO: Bezugsgröße in der Optimierung prüfen.
   * - GuD – fixe Betriebskosten
     - 29.300 €/MW/a
     - [DEA2020]_
     - HTWP: elektrische Nennleistung; TODO: Bezugsgröße in der Optimierung prüfen.
   * - GuD – variable Betriebskosten
     - 4,40 €/MWh
     - [DEA2020]_
     - Elektrische Arbeit.
   * - BHKW – Investitionskosten
     - 950.000 €/MW
     - [DEA2020]_
     - HTWP: Biogas-BHKW, elektrische Nennleistung; TODO: Übertragbarkeit und Bezugsgröße prüfen.
   * - BHKW – fixe Betriebskosten
     - 9.750 €/MW/a
     - [DEA2020]_
     - HTWP: Biogas-BHKW, elektrische Nennleistung; TODO: Bezugsgröße prüfen.
   * - BHKW – variable Betriebskosten
     - 7,50 €/MWh
     - [DEA2020]_
     - Elektrische Arbeit; HTWP: Biogas-BHKW.
   * - Gaskessel – Investitionskosten
     - 60.000 €/MW
     - [DEA2020]_
     - Thermische Nennleistung.
   * - Gaskessel – fixe Betriebskosten
     - 1.950 €/MW/a
     - [DEA2020]_
     - Thermische Nennleistung.
   * - Gaskessel – variable Betriebskosten
     - 1,10 €/MWh
     - [DEA2020]_
     - Erzeugte Wärme; ohne zusätzliche Brennstoffkosten.
   * - Elektrodenheizkessel – Investitionskosten
     - 150.000 €/MW
     -
     - **TODO** – Thermische Nennleistung.
   * - Elektrodenheizkessel – fixe Betriebskosten
     - 1.070 €/MW/a
     -
     - **TODO** – Thermische Nennleistung.
   * - Elektrodenheizkessel – variable Betriebskosten
     - 0,90 €/MWh
     -
     - **TODO** – Erzeugte Wärme.
   * - Solarthermie – Investitionskosten
     - 199 €/m²
     -
     - **TODO** – Kollektorfläche. HTWP verwendet eine andere, affine Kostenfunktion nach [Brandt2019]_ und [Pehnt2017]_.
   * - Solarthermie – fixe Betriebskosten
     - 0,04
     -
     - **TODO** – UI: €/MWh; in der Kostenfunktion auf die Kollektorfläche bezogen. Einheit und Herkunft klären; HTWP: 0,09 €/MWh.
   * - Solarthermie – variable Betriebskosten
     - 0,22 €/MWh
     -
     - **TODO** – Erzeugte Wärme; HTWP: 0,21 €/MWh.
   * - Wärmespeicher – Investitionskosten
     - 580 €/MWh
     -
     - **TODO** – Speicherkapazität. HTWP verwendet eine andere, affine Kostenfunktion nach [Pehnt2017]_.
   * - Wärmespeicher – fixe Betriebskosten
     - 3,00 €/MWh/a
     - [DEA2018]_
     - Bezogen auf die Speicherkapazität.
   * - Wärmespeicher – variable Betriebskosten
     - 0,01 €/MWh
     - [DEA2018]_
     - Im Modell bei Be- und Entladung angesetzt.
   * - Externe Wärmequelle – Investitionskosten
     - 0 €/MW
     -
     - **TODO** – Keine Investitionskosten im Ausgangszustand.
   * - Externe Wärmequelle – fixe Betriebskosten
     - 0 €/MW/a
     -
     - **TODO** – Keine fixen Betriebskosten im Ausgangszustand.
   * - Externe Wärmequelle – variable Betriebskosten
     - 50 €/MWh
     -
     - **TODO** – Kosten des externen Wärmebezugs.

Für alle Anlagentypen sind die relativen Investitionskostenförderungen (``inv_bonus_rel``) und Betriebskostenförderungen (``op_cost_bonus_rel``) auf 0 % voreingestellt. Dies ist eine interne, förderneutrale Modellvorgabe; die Anwendung spezifischer Förderrichtlinien muss durch die Nutzenden selbst geprüft werden.

Wärmenetz
=========

Für das Wärmenetz können spezifische Netzkosten, jährliche bzw. einmalige Gesamtkosten oder keine Netzkosten angesetzt werden. Bei spezifischen Kosten werden die Investitionskosten und fixen Betriebskosten aus der Trassenlänge ermittelt; die variablen Kosten ergeben sich aus der Wärmelast. Die vorgegebenen Werte sind nicht als projektspezifische Kostenschätzung zu verstehen.

.. list-table::
   :header-rows: 1
   :widths: 29 20 19 32

   * - Parameter / Datensatz
     - Standardwert / Zeitraum
     - Quelle
     - Erläuterung
   * - Kalkulationsmethode (``calc_network``)
     - Spezifische Kosten
     - Interne Modell-/Bedienvorgabe
     - Alternative: Gesamtkosten oder keine Netzkosten.
   * - Trassenlänge (``net_dist``)
     - 10 km
     -
     - **TODO** – Beispielwert.
   * - Spezifische Investitionskosten (``net_inv_spez``)
     - 1.500 €/m
     -
     - **TODO** – Bezogen auf die Trassenlänge.
   * - Spezifische fixe Betriebskosten (``net_op_cost_fix``)
     - 50 €/m/a
     -
     - **TODO** – Bezogen auf die Trassenlänge.
   * - Spezifische variable Betriebskosten (``net_op_cost_var``)
     - 1,50 €/MWh
     -
     - **TODO** – Bezogen auf die bereitgestellte Wärmemenge.
   * - Gesamtinvestitionskosten (``net_inv_total``)
     - 1.000.000 €
     -
     - **TODO** – Alternativer pauschaler Ansatz.
   * - Fixe Betriebskosten gesamt (``net_op_cost_fix_total``)
     - 70.000 €/a
     -
     - **TODO** – Alternativer pauschaler Ansatz.
   * - Variable Betriebskosten gesamt (``net_op_cost_var_total``)
     - 30.000 €/a
     -
     - **TODO** – Alternativer pauschaler Ansatz.

Energiewirtschaftliche Rahmenbedingungen
========================================

.. list-table::
   :header-rows: 1
   :widths: 29 20 19 32

   * - Parameter / Datensatz
     - Standardwert / Zeitraum
     - Quelle
     - Erläuterung
   * - Wärmeerlös (``heat_price``)
     - 76,74 €/MWh
     - [AGFW2020]_; [HTWP2024]_
     - Mittelwert der drei Mischpreise 79,04, 76,93 und 74,24 €/MWh bei 1.800 Jahresnutzungsstunden; historischer Referenzwert. Bericht nennt 2019, Literaturdatenbank Stichtag 01.10.2020.
   * - Zusätzliche Strompreisbestandteile Netzbezug (``elec_consumer_charges_grid``)
     - 52,52 €/MWh
     - [HTWP2024]_
     - Pauschaler JSON-Startwert entspricht HTWP-Referenz 2019 ohne EEG-Umlage; wird bei aktiver Jahrestabelle neu berechnet.
   * - Zusätzliche Strompreisbestandteile Eigennutzung (``elec_consumer_charges_self``)
     - 20,50 €/MWh
     - [HTWP2024]_
     - Pauschaler JSON-Startwert entspricht HTWP-Referenz 2019 ohne EEG-Umlage; wird bei aktiver Jahrestabelle neu berechnet.
   * - Energiesteuer Erdgas (``energy_tax``)
     - 5,50 €/MWh
     - [EnergieStG]_; [HTWP2024]_
     - Historischer Steueransatz für Erdgas gemäß § 2 Abs. 3 EnergieStG; aktuelle Anwendbarkeit prüfen.
   * - Vermiedene Netznutzungsentgelte (``vNNE``)
     - 7,00 €/MWh
     - [ENKF2019]_; [SolWW2022]_; [HTWP2024]_
     - Historische Modellannahme der Vorgängerprojekte, kein allgemeingültiger Anspruch.
   * - Kapitalzins (``capital_interest``)
     - 5 %
     -
     - **TODO** – Kalkulationszins für die wirtschaftliche Bewertung.
   * - Betrachtungsdauer (``lifetime``)
     - 20 Jahre
     -
     - **TODO** – Zeithorizont für die Annuitätenberechnung.

Historische Strompreisbestandteile
----------------------------------

In ``param_bound.json`` sind für 2014 bis 2024 folgende Bestandteile jeweils für Netzbezug und Eigennutzung hinterlegt: EEG-Umlage, Netzentgelte, KWKG-Umlage, Offshore-Haftungsumlage, AbLaV-Umlage, StromNEV-Umlage, Konzessionsabgabe und Stromsteuer. Die Werte können in der Benutzeroberfläche angepasst werden. Die Summe der Einzelwerte in ct/kWh wird mit dem Faktor 10 in €/MWh umgerechnet. Die Berücksichtigung der Preisbestandteile ist standardmäßig aktiviert; bei Deaktivierung werden beide Zuschläge auf null gesetzt.

Die HTWP-Referenz für 2019 stützt sich auf [AgoraPtH2014]_, [BDEW2021]_ und [BNetzA2016]_ sowie die damals geltenden gesetzlichen Regelungen (u. a. EEG, StromNEV, EnWG, KWKG und AbLaV). Die übrigen historischen Jahreswerte sind noch mit konkreten Quellen zu hinterlegen. Die Tabelle zeigt die tatsächlich im OWP-Tool hinterlegten Einzelwerte, nicht eine aktuelle rechtliche Bewertung.

.. list-table::
   :header-rows: 1

   * - Jahr
     - EEG
     - Netz
     - KWKG
     - Offshore
     - AbLaV
     - StromNEV
     - Konzession
     - Stromsteuer
     - EEG eigen
     - Quelle
     - Status
   * - 2014
     - 6,24
     - 1,9
     - 0,055
     - 0,058
     - 0,009
     - 0,066
     - 0,11
     - 2,05
     - 1,872
     -
     - **TODO**
   * - 2015
     - 6,17
     - 2,12
     - 0,051
     - 0,046
     - 0,006
     - 0,057
     - 0,11
     - 2,05
     - 1,851
     -
     - **TODO**
   * - 2016
     - 6,354
     - 2,06
     - 0,06
     - 0,03
     - 0
     - 0,06
     - 0,11
     - 2,05
     - 2,224
     -
     - **TODO**
   * - 2017
     - 6,88
     - 2,26
     - 0,095
     - 0,035
     - 0,006
     - 0,064
     - 0,11
     - 2,05
     - 2,752
     -
     - **TODO**
   * - 2018
     - 6,792
     - 2,36
     - 0,168
     - 0,049
     - 0,011
     - 0,063
     - 0,11
     - 2,05
     - 2,717
     -
     - **TODO**
   * - 2019
     - 6,405
     - 2,33
     - 0,28
     - 0,416
     - 0,005
     - 0,061
     - 0,11
     - 2,05
     - 2,562
     - [HTWP2024]_
     -
   * - 2020
     - 6,756
     - 2,7
     - 0,226
     - 0,416
     - 0,0075
     - 0,063
     - 0,12
     - 2,05
     - 2,702
     -
     - **TODO**
   * - 2021
     - 6,5
     - 2,67
     - 0,254
     - 0,395
     - 0,009
     - 0,064
     - 0,11
     - 2,05
     - 2,6
     -
     - **TODO**
   * - 2022
     - 1,862
     - 2,96
     - 0,378
     - 0,419
     - 0,003
     - 0,066
     - 0,11
     - 2,05
     - 0,745
     -
     - **TODO**
   * - 2023
     - 0
     - 3,35
     - 0,378
     - 0,591
     - 0
     - 0,066
     - 0,11
     - 2,05
     - 0
     -
     - **TODO**
   * - 2024
     - 0
     - 4,12
     - 0,28
     - 0,66
     - 0
     - 0,08
     - 0,19
     - 2,05
     - 0
     -
     - **TODO**

Alle Tabellenwerte in ct/kWh. Bei Eigennutzung sind neben der in der Spalte „EEG eigen“ genannten EEG-Umlage ausschließlich 2,05 ct/kWh Stromsteuer hinterlegt; die übrigen sechs Bestandteile sind jeweils null. Für die Stromsteuer ist die gesetzliche Grundlage historisch im StromStG zu finden. Die übrigen Jahreswerte einschließlich der Eigenverbrauchsannahmen bedürfen der Quellenprüfung.

Optimierungsparameter und weitere Bedienvorgaben
================================================

.. list-table::
   :header-rows: 1
   :widths: 29 20 19 32

   * - Parameter / Datensatz
     - Standardwert / Zeitraum
     - Quelle
     - Erläuterung
   * - Solver (``Solver``)
     - HiGHS
     - Interne Modell-/Bedienvorgabe
     - Standardmäßig ausgewählter Solver; alternativ Gurobi oder SCIP.
   * - Relative MIP-Lücke (``MIPGap``)
     - 2 %
     - Interne Modell-/Bedienvorgabe
     - Abbruchtoleranz der gemischt-ganzzahligen Optimierung.
   * - Zeitbegrenzung (``TimeLimit``)
     - Deaktiviert; bei Aktivierung 10 min
     - Interne Modell-/Bedienvorgabe
     - JSON-Startwert 600 s; ohne aktivierte Begrenzung wird None übergeben.
   * - Anzahl je ausgewähltem Anlagentyp
     - 1
     - Interne Modell-/Bedienvorgabe
     - Initiale Anzahl der Anlagen; mehrere Anlagen desselben Typs sind möglich.
   * - Wärmelastdatensatz / Jahr
     - Flensburg / letztes verfügbares Jahr
     - Interne Modell-/Bedienvorgabe
     - Standardauswahl der Wärmelast.
   * - Einstrahlungsstandort
     - Schleswig
     - Interne Modell-/Bedienvorgabe
     - Standardauswahl für Solarthermie.
   * - Jahr der Versorgungsdaten und Preisbestandteile
     - Jahr der Wärmelast, sonst 2024
     - Interne Modell-/Bedienvorgabe
     - Startjahr für historische Strom-, Gas-, CO₂- und Einstrahlungsdaten sowie Emissionsfaktoren und Strompreisbestandteile.
   * - Preis- und Emissionsvarianten
     - Variabel
     - Interne Modell-/Bedienvorgabe
     - Historische Zeitreihen als Ausgangsauswahl, alternativ konstant oder eigene Daten.
   * - Zeitraumauswahl
     - Ganzes Jahr; bei Aktivierung 28.03.–02.07.
     - Interne Modell-/Bedienvorgabe
     - Voreinstellung der optionalen exakten Datumsauswahl.
   * - Skalierung
     - Deaktiviert
     - Interne Modell-/Bedienvorgabe
     - Historische Zeitreihen werden zunächst unverändert verwendet.
   * - Skalierungsfaktor / Stauchungsfaktor
     - 1,0 / 1,0
     - Interne Modell-/Bedienvorgabe
     - Bei einfacher Skalierung unveränderte Werte; erweiterte Skalierung verändert die Abweichungen vom Median.
   * - Offset bei erweiterter Skalierung
     - 1,0 in der jeweiligen Einheit
     - Interne Modell-/Bedienvorgabe
     - Wird zu der um den Median skalierten Zeitreihe addiert; bei unveränderter Zeitreihe ist 0 zu wählen.
   * - Haushaltszahl / Gesamtwärmelast als Skalierungsziel
     - Referenzhaushalte / Summe des ausgewählten Lastprofils
     - Interne Modell-/Bedienvorgabe
     - Startwert ergibt sich aus den obigen Referenzzahlen bzw. aus dem gewählten Datensatz.

Die genannten Bedienvorgaben sind im Programmcode hinterlegt und keine externen Datensätze. Importierte eigene Energiesysteme bzw. eigene Zeitreihen werden von den Nutzenden bereitgestellt; ihre Herkunft kann das OWP-Tool nicht automatisch belegen. Platzhalter bei noch nicht ausgewählten Uploads sind keine fachlichen Standarddatensätze.

Quellenverzeichnis
==================

.. [HTWP2024] Freißmann, J.; Fritz, M.; Tuschy, I. (2024): Hochtemperaturwärmepumpen in der Nah- und Fernwärmeversorgung – Technologieperspektive für den kurz- und mittelfristigen Einsatz in multivalenten Systemen. ZNES. https://doi.org/10.5281/zenodo.13311533

.. [FL2016] Stadtwerke Flensburg GmbH (2019): District heating network data for the city of Flensburg from 2014 to 2016. Zenodo. https://doi.org/10.5281/zenodo.2553968

.. [FL2019] Freißmann, J.; Fritz, M.; Tuschy, I.; Stadtwerke Flensburg GmbH (2024): Network Data of the District Heating System for the city of Flensburg from 2017–2019. Zenodo. https://doi.org/10.5281/zenodo.10508280

.. [FL2024] Freißmann, J.; Fritz, M.; Tuschy, I.; Stadtwerke Flensburg GmbH (2025): Network Data of the District Heating System for the city of Flensburg from 2020–2024. Version 1.0.0, Zenodo. https://doi.org/10.5281/zenodo.17177421

.. [SO2019] Sønderborg Varme A/S (2023): Network Data of the District Heating System for the city of Sønderborg from 2016–2019. Zenodo. https://doi.org/10.5281/zenodo.7972964

.. [DWDTemp] Deutscher Wetterdienst: Climate Data Center (CDC), Hourly Climate Observations Germany. https://opendata.dwd.de/climate_environment/CDC/observations_germany/climate/hourly/

.. [Hellwig2003] Hellwig, M. (2003): Entwicklung und Anwendung parametrisierter Standard-Lastprofile. Dissertation. https://nbn-resolving.de/urn/resolver.pl?urn:nbn:de:bvb:91-diss2003100715846

.. [Agora] Agora Energiewende: Agorameter – historische Daten zur Stromerzeugung, zum Stromverbrauch und zu Strompreisen. https://www.agora-energiewende.de/daten-tools/agorameter

.. [EEX2021] European Energy Exchange (EEX): EGIX Erdgaspreisindex der PEGAS. Historische Literaturangabe im HTWP-Bericht: 2010–2020. https://www.hansewerk-natur.com/de/service/downloads/preisindizes.html

.. [AgoraDoku] Agora Energiewende (2020): Agorameter – Dokumentation, Version 10. https://www.agora-energiewende.de/veroeffentlichungen/agorameter-dokumentation/

.. [ENTSOE] ENTSO-E: Transparency Platform. https://transparency.entsoe.eu/

.. [AGEB] AG Energiebilanzen e. V. (2022): Stromerzeugung nach Energieträgern, 1990–2021. https://ag-energiebilanzen.de/daten-und-fakten/zusatzinformationen/

.. [Juhrich2016] Juhrich, K. (2016): CO₂-Emissionsfaktoren für fossile Brennstoffe. Umweltbundesamt.

.. [UBA2023] Umweltbundesamt (2023): Entwicklung der spezifischen Treibhausgas-Emissionen des deutschen Strommix in den Jahren 1990–2022. Climate Change 20/2023. https://www.umweltbundesamt.de/sites/default/files/medien/1410/publikationen/2023_05_23_climate_change_20-2023_strommix_bf.pdf

.. [DWDStrahlung] Deutscher Wetterdienst: Regenerative Energien – Globalstrahlung: Summe aus direkter und diffuser Sonnenstrahlung. https://www.dwd.de/DE/leistungen/solarenergie/solarenergie.html

.. [DIN5034] DIN 5034-2:1985-02: Tageslicht in Innenräumen – Grundlagen. https://doi.org/10.31030/1114021

.. [Quaschning2015] Quaschning, V. (2015): Regenerative Energiesysteme – Technologie, Berechnung, Simulation.

.. [SolarKeymark2018] SPF Institut für Solartechnik (2018): Solar Keymark Certificate – Arcon-Sunmark HT-SolarBoost 35/10. http://www.solarkeymark.nl/DBF/PDF_Downloads/DS_579.pdf

.. [DEA2020] Energinet.dk / Danish Energy Agency (2016, Update April 2020): Technology Data – Energy Plants for Electricity and District Heating Generation. https://ens.dk/sites/ens.dk/files/Statistik/technology_data_catalogue_for_el_and_dh_-_0009.pdf

.. [DEA2018] Energinet.dk (2018): Technology Data for Energy Storage. https://ens.dk/en/our-services/projections-and-models/technology-data/technology-data-energy-storage

.. [Konstantin2013] Konstantin, P. (2013): Praxisbuch Energiewirtschaft – Energieumwandlung, -transport und -beschaffung im liberalisierten Markt. 3. Aufl., Springer. ISBN 978-3-642-37265-0.

.. [AGFW2020] Arbeitsgemeinschaft Fernwärme (AGFW) e. V. (2020): Fernwärme-Preisübersicht (Stichtag: 01.10.2020). https://www.agfw.de/energiewirtschaft-recht-politik/wirtschaft-und-markt/markt-preise/preisanpassung

.. [EnergieStG] Energiesteuergesetz (EnergieStG), § 2 Abs. 3. https://www.gesetze-im-internet.de/energiestg/__2.html

.. [ENKF2019] Kaldemayer, C.; Boysen, C.; Tuschy, I. (2019): Elektrizitätsnetzgekoppelte Fernwärmeversorgung 2020 (ENKF). ZNES.

.. [SolWW2022] Freißmann, J.; Fritz, M.; Tuschy, I. (2022): Solarthermisch gestützte Nah- und Fernwärmeversorgung als Baustein der sektorgekoppelten Wärmewende in Schleswig-Holstein (SolWW-SH). http://www.znes-flensburg.de/sites/default/files/projects/SolWW-SH_ZNES_Forschungsergebnisse_2020531.pdf

.. [AgoraPtH2014] Agora Energiewende (2014): Power-to-Heat zur Integration von ansonsten abgeregeltem Strom aus Erneuerbaren Energien. https://static.agora-energiewende.de/fileadmin/Projekte/2013/power-to-heat/Agora_PtH_Langfassung_WEB.pdf

.. [BDEW2021] BDEW (2021): Strompreisanalyse Januar 2021. https://www.bdew.de/media/documents/BDEW-Strompreisanalyse_no_halbjaehrlich_Ba_online_28012021.pdf

.. [BNetzA2016] Bundesnetzagentur (2016): Monitoringbericht 2016. https://www.bundesnetzagentur.de/SharedDocs/Mediathek/Monitoringberichte/Monitoringbericht2016.pdf?__blob=publicationFile&v=2

.. [Brandt2019] Brandt, M. (2019): Untersuchung zum Einsatz von Solarthermie in multivalenten Wärmeversorgungsnetzen. Masterarbeit, Hochschule Flensburg (unveröffentlicht).

.. [Pehnt2017] Pehnt, M. et al. (2017): Wärmenetzsysteme 4.0: Endbericht – Kurzstudie zur Umsetzung der Maßnahme: Modellvorhaben erneuerbare Energien in hocheffizienten Niedertemperaturwärmenetzen. https://www.ifeu.de/wp-content/uploads/W%C3%A4rmenetze-4.0-Endbericht-final.pdf

Dynamo26 3.0


Führe DYNAMO26 Code v1.0 vollständig aus.

Eingabe:
- Verwende die hochgeladene Prior-State-TXT als prior_state_file.
- Behandle die Prior-State-TXT ausschließlich als gespeicherten Vorzustand.
- Verwende sie niemals als aktuellen Quellenlauf.
- Führe einen neuen aktuellen Quellenlauf aus.
- Suche nur nach neuen modellrelevanten Ereignissen nach created_at_run_date des Prior-State.

Pflichtablauf:
1. INIT_RUN
2. LOAD_MODEL
3. LOAD_PRIOR_STATE
4. BUILD_SOURCE_PLAN
5. FETCH_CURRENT_SOURCES
6. CLASSIFY_SOURCES
7. AUDIT_SOURCE_QUALITY
8. AUDIT_BIAS
9. DETERMINE_OUTPUT_STATUS
10. RESOLVE_ENTITIES
11. EXTRACT_EVENT_SIGNALS
12. CHECK_INPUT_COMPLETENESS
13. NORMALIZE_SIGNALS
14. BUILD_TEAM_STATES
15. APPLY_EVENT_UPDATES
16. RECALCULATE_INDICES
17. CALCULATE_DYNAMIC_STRENGTH
18. BUILD_UPCOMING_MATCH_PLAN
19. BUILD_PRE_MATCH_SOURCE_PLAN
20. FETCH_PRE_MATCH_SOURCES
21. CLASSIFY_SOURCES für den Pre-Match-Quellenlauf
22. AUDIT_SOURCE_QUALITY für den Pre-Match-Quellenlauf
23. BUILD_PRE_MATCH_DOSSIERS
24. AUDIT_PRE_MATCH_DOSSIERS
25. BUILD_MATCHUP_MATRIX
26. SIMULATE_PATHS
27. NORMALIZE_PROBABILITIES
28. ATTACH_PROBABILITIES
29. CALCULATE_CRASH_RISK
30. GENERATE_CAUSAL_CHAINS
31. AUDIT_MODEL
32. SELF_CORRECT falls nötig
33. RENDER_OUTPUT oder RENDER_NOT_CURRENT
34. BUILD_CHANGE_SUMMARY
35. EXPORT_ARTIFACTS
36. AUDIT_ARTIFACTS
37. DELIVER

Harte Regeln:
- Ohne aktuellen Quellenlauf keine aktuelle Prognose.
- Prior-State ist keine Quelle.
- Keine Wahrscheinlichkeit ohne bestandenen Quellenlauf, Pfadsimulation und Audit.
- Keine Änderung ohne vollständige New-Event-Kette:
  Quelle -> Tier -> Datum -> Ereignis nach created_at_run_date -> Entität -> Signaltyp -> Indexwirkung -> Modellmechanismus -> Wahrscheinlichkeitseffekt.
- Wenn keine belastbaren neuen modellrelevanten Ereignisse gefunden werden:
  output_status = CURRENT
  run_result_type = CURRENT_WITH_NO_NEW_MODEL_EVENTS
  Zustand unverändert lassen.
  Keine künstlichen Deltas erzeugen.
- Fehlende Premiumdaten/xG/Pressing/Set-Piece-Daten nicht automatisch als NOT_CURRENT behandeln.
- Social Media ohne Bestätigung darf keinen Fakt setzen.
- T7 bleibt als Modellinput vollständig verboten: keine externen Prognosen, Projektionen, Buchmacherquoten, Elo-/Ranking-Prognosen oder Medien-Wahrscheinlichkeiten gezielt suchen, zitieren, speichern, kalibrieren oder kausal verwenden. Wenn eine ansonsten zulässige T0-T6-Quelle zufällig einen T7-Abschnitt enthält, wird ausschließlich dieser T7-Teil als ignored_t7_segment dokumentiert und ignoriert; die unabhängig belegten Nicht-T7-Fakten derselben Quelle dürfen weiter verwendet werden, sofern sie zusätzlich durch zulässige Quellenqualität/Audit gedeckt sind. Ein ignorierter T7-Abschnitt darf keinen NOT_CURRENT-Status auslösen und keine Kausalkette, Indexwirkung, Wahrscheinlichkeit oder Formulierung beeinflussen.
- Gruppenabschluss- und Bracket-Regel: Jeder Lauf prüft zwei Bedingungen: (1) Ist die Gruppenphase laut offiziellem Turnierkern vorbei? (2) Enthält der Prior-State bereits group_stage_final_qualification_gate_passed=true mit vollständigen Listen qualified_teams, best_third_placed_teams und eliminated_teams sowie einem klar strukturierten knockout_bracket_state? Nur wenn (1)=true und (2)=false/fehlt, muss der Lauf das Gruppenabschluss-/K.o.-Qualifikations-Gate vollständig ausführen. Danach muss der neue Prior-State group_stage_final_qualification_gate_passed=true, die finalen Listen und knockout_bracket_state speichern. Der gespeicherte Turnierbaum muss im eigenen Abschnitt KNOCKOUT_BRACKET_STATE maschinenlesbar/YAML-ähnlich und ohne reine Prosabeschreibung stehen: current_stage, bracket_locked, rounds mit match_id, round, slot, team_a, team_b, winner_to, loser_eliminated, status, winner, score, sources sowie team_bracket_positions mit half, quarter, slot und current_match_id je Team. In späteren Läufen wird dieser Baum aus dem Prior-State übernommen und nur durch offiziell bestätigte neue Ergebnisse, Korrekturen oder Bracket-Änderungen aktualisiert. Vor bestandenem Gate dürfen Drittplatzierte nicht endgültig als qualifiziert/ausgeschieden behandelt werden.
- Bracket-basierte Pfadwahrscheinlichkeiten: Sobald knockout_bracket_state vorhanden ist, müssen quarter_final_probability_percent, semi_final_probability_percent, final_probability_percent und title_probability_percent aus demselben festen Turnierbaum bzw. derselben Pfadsimulation berechnet werden. Für jedes Team gilt zwingend: 0.0 <= title_probability_percent <= final_probability_percent <= semi_final_probability_percent <= quarter_final_probability_percent <= 100.0. Für jede feste Paarung gilt für die nächste Runde: Summe der beiden noch lebenden Teams = 100.0, z. B. R16-Paarung -> QF-Summe 100.0, QF-Paarung -> HF-Summe 100.0, HF-Paarung -> Final-Summe 100.0, Finale -> Title-Summe 100.0. Diese Paarungssumme darf nicht auf Title % übertragen werden, solange es nicht das Finale ist; zwei Viertelfinalteams können gemeinsam deutlich weniger als 100.0 Title % besitzen, weil spätere Rundenpfade erst danach wirken. Teams im selben Viertelbaum können nicht beide dasselbe Halbfinale erreichen; Teams im selben Halbbaum können nicht beide das Finale erreichen. Title % bleibt global normalisiert auf 100.0 und darf je Baumhälfte ungleich verteilt sein; eine Baumhälfte darf also mehr als 50% Titelchance tragen, wenn Stärke und Pfad das rechtfertigen.
- Ausblick auf kommende Spiele: Nach der vollständigen Ergebnis- und Bracket-Prüfung muss für jede bereits feststehende kommende Partie ein eigenes aktuelles PRE_MATCH_DOSSIER recherchiert werden. Prüfe mindestens offiziellen Termin/Ort/Anstoßzeit und Schiedsrichterstatus, Fitness/Verfügbarkeit/Sperren, erwartete Aufstellungen, belastbare Aussagen aus Training oder Pressekonferenzen, Teamumfeld/Stimmung, Spielweise beider Teams und konkrete taktische Wechselwirkungen, Ruhezeit/Reise sowie Wetter zur Anstoßzeit. Nutze T0 bis T5 entsprechend ihrer Quellenfunktion; Social Media bleibt T6 warning_only und T7 bleibt als Modellinput verboten. Nicht bestätigte oder noch nicht verfügbare Angaben müssen sichtbar als unbekannt markiert werden. Der informative Ausblick in beiden HTML-Dateien muss jede kommende Partie einzeln, in der jeweiligen Ausgabesprache und mit direkt zugeordneten Quellenlinks darstellen. Zusätzlich muss jedes PRE_MATCH_DOSSIER eine eigene interne Matcheinschätzung enthalten: bedingte Siegchance Team A/Team B, Wahrscheinlichkeit für Verlängerung, Wahrscheinlichkeit für Elfmeterschießen, wichtigste Spielverlaufsszenarien, wem eine frühe Führung stärker hilft, wer nach frühem Rückstand besser reagieren kann, welches Team aktiver auf Tore spielt und welches eher auf Kontrolle, Risikoarmut oder notfalls Elfmeterschießen vertraut. Diese internen Matchquoten sind Dynamo26-Modellwerte und dürfen nicht aus externen Wettquoten, Medienprognosen oder T7-Quellen kalibriert werden.
- Geringe Pre-Match-Modellwirkung: Kurzfristige Erkenntnisse aus PRE_MATCH_DOSSIERS dürfen die grundlegende Teamstärke und den Turnierpfad nur geringfügig verändern. Die Wirkung erfolgt als stark gedämpfter, confidence-gewichteter Matchup-Kontext auf der bedingten Siegchance der konkreten Paarung, nicht als feste absolute Prozentpunktgrenze, nicht als dauerhafte Überschreibung der Teamstärke und nicht direkt als pauschales Titelprozent-Delta. Eine Wahrscheinlichkeitsänderung braucht weiterhin eine vollständige New-Event-Kette nach dem Prior-Cutoff; bereits bekannte und unveränderte Spielstile oder Kontextinformationen dürfen den Ausblick erläutern, aber kein neues Delta erzeugen. Eine nach dem Cutoff neu feststehende Paarung oder neu bestätigte Availability-, Aufstellungs-, Schiedsrichter-, Reise- oder Wetterinformation kann eine solche Kontextkette auslösen. Bei einer K.-o.-Paarung muss jede kontextbedingte Erhöhung der bedingten Siegchance eines Teams exakt als gleich große Verringerung beim Gegner erscheinen; nur die konkrete Paarungssumme der nächsten Runde bleibt bei 100.0. Titelwahrscheinlichkeiten werden danach aus derselben Pfadsimulation abgeleitet und müssen nicht paarweise 100.0 ergeben: In einem Viertelfinale können beide Teams zusammen z. B. nur rund 20% Titelchance tragen, weil Halbfinal-, Final- und Titelpfade nachgelagert wirken. Nur im Finale entspricht die bedingte Matchsiegchance direkt der Title-%-Summe 100.0. Der matchspezifische Pre-Match-Faktor muss zur aktuellen bedingten Matchquote passen: close matchups erhalten wegen quote_sensitivity = 4 * p * (1 - p) mehr sichtbaren Spielraum, klare Favoriten weniger; der Log-Odds-Kontext bleibt pro Paarung auf maximal 0.08 begrenzt und wird zusätzlich nach Quellenvertrauen und Signalqualität gedämpft. Bei Gruppenspielen ist eine Modellwirkung nur zulässig, wenn die aktuelle Tabellen-, Qualifikations- oder Rotationskonstellation einen konkreten Mechanismus erzeugt; reine Stimmung, allgemeine Erwartungen oder bekannte Spielstile ohne konstellationsbezogenen Effekt dürfen dort keine Wahrscheinlichkeit verschieben. Schiedsrichter- und Stimmungsinformationen sind besonders vorsichtig zu behandeln und dürfen nur bei mehreren belastbaren Anhaltspunkten bzw. klar belegter stilistischer Relevanz einen sehr kleinen Effekt erhalten. Keine Doppelzählung desselben Signals in Teamindizes, Matchup-Kontext und Crash-Risk.
- Tabellen-Lebenszyklus ab dem nächsten Tageslauf: Teams mit title_probability 0.0 dürfen nur dann in der Haupt-Wahrscheinlichkeitstabelle in HTML und TITLE_PROBABILITY_PRIOR_TABLE erscheinen, wenn sie im aktuellen Lauf gerade auf 0.0 gefallen bzw. gerade ausgeschieden sind. Teams, die bereits in einem früheren Lauf auf 0.0 gefallen waren, werden nicht weiter als Haupttabellenzeile fortgeführt, müssen aber im Prior-State in eliminated_teams, knockout_eliminated_teams_after_prior_cutoff oder einem Archiv-/Statusabschnitt nachvollziehbar erhalten bleiben. Wenn Restfeld eine positive Titelwahrscheinlichkeit hat, muss die Haupttabelle genau 25 Zeilen enthalten. Freie Tabellenplätze müssen mit noch aktiven Teams gefüllt werden, die fachlich im Restfeld-Aggregat enthalten sind; ausgeschiedene Restfeld-Teams dürfen nicht als Füllteams erscheinen. Die Füllteams sind nach internem Restfeld-Wert absteigend auszuwählen. Wenn diese Werte im Prior-State nicht bestimmt sind, muss der Lauf sie aus Restfeld-Signalen, K.o.-Pfad, Qualifikationsstatus, Gegnerpfad, Teamstärke-/Indexhinweisen, aktuellem Turnierbaum und aktuellen Quellen nachvollziehbar ermitteln. Restfeld-Füllteams sind keine reinen Anzeigezeilen: Wenn sie in der Haupttabelle erscheinen, müssen für sie title_probability_percent, final_probability_percent, die aktuell angezeigte Zwischenrundenwahrscheinlichkeit, early_exit_probability_percent und crash_risk_score vollständig modelliert werden. Dasselbe gilt für eine verbleibende Restfeld-Restzeile als Aggregat. Dabei dürfen keine Werte erfunden oder doppelt gezählt werden: Die Summe der Restfeld-Füllteam-Titelwerte plus Restfeld-Restzeile muss exakt der Restfeld-Gesamtwahrscheinlichkeit entsprechen. Die Final-/Zwischenrunden-/Early-Exit-/Crash-Werte müssen aus derselben Pfadsimulation bzw. derselben Restfeld-Komponentenrechnung stammen; wenn das nicht möglich ist, gilt der Lauf als nicht erfolgreich und darf nicht finalisiert werden. Wenn Restfeld-Komponenten neu aus dem Aggregat gehoben werden, muss geprüft werden, ob die Pfadsimulation dadurch andere Teams beeinflusst; nötige Korrekturen an anderen Teams müssen mit Kausalkette dokumentiert werden. Die Restfeld-Restzeile steht immer am Ende der Haupttabelle, solange ein positiver Restfeld-Rest verbleibt. Wenn kein positives Restfeld mehr existiert, wird keine Restfeld-Zeile angezeigt und die Haupttabelle darf weniger als 25 Zeilen haben.
- Summen- und Anzeige-Regeln für Wahrscheinlichkeitstabellen: title_probability_percent muss über die gesamte Haupttabelle inklusive Restfeld-Komponenten und Restfeld-Rest exakt 100.0 ergeben. final_probability_percent muss exakt 200.0 ergeben, solange mindestens Halbfinale oder frühere Runden offen sind; wenn nur noch das Finale offen ist, bilden die beiden Finalisten zusammen 200.0 Final %. Die Zwischenrunden-Spalte ist phasenabhängig und exklusiv: Vor bzw. während der Achtelfinal-Entscheidung wird ausschließlich QF % angezeigt und summiert auf 800.0. Sobald alle Achtelfinals beendet sind und der Viertelfinalpfad die nächste relevante Zwischenrunde ist, wird ausschließlich HF %/Semi-final % angezeigt und QF % vollständig aus der HTML-Haupttabelle entfernt. Sobald alle Halbfinals beendet sind und nur noch das Finale offen ist, wird auch HF %/Semi-final % vollständig entfernt; sichtbar bleiben nur Final % und Title %. Frühere Rundenwerte bleiben für Audit und Verlustfreiheit im Prior-State erhalten, dürfen aber nicht zusätzlich als überholte HTML-Spalten erscheinen. Diese Summenregeln gelten auch dann, wenn ein Restfeld-Rest als Aggregatzeile verbleibt; die Aggregatzeile trägt dann die Summe der in ihr enthaltenen Mikro-Pfade für Title, Final und die aktuell angezeigte Zwischenrunde. crash_risk_score ist dagegen ein individueller Risiko-Score und darf nicht aufsummiert werden. early_exit_probability_percent ist ebenfalls kein sinnvoller Summenwert für eine Restfeld-Rest-Aggregatzeile. Für eine Restfeld-Rest-Aggregatzeile müssen crash_risk_score und early_exit_probability_percent leer bleiben, solange sie nicht ausdrücklich als gewichtete Aggregatindikatoren mit eigener Kennzeichnung definiert werden. Wenn die Summenregeln nach Rundung nicht erfüllt sind oder ein Team title > final > HF > QF verletzt, muss der Lauf nachjustieren oder als nicht erfolgreich gelten.
- Jede OUTPUT-Aussage braucht eine Kausalkette.
- Die kurze Changes-TXT muss einsprachig englisch und strukturiert sein, im Format:
  DYNAMO26 changes - YYYY-MM-DD
  Status: CURRENT|DEGRADED. New verified matches after prior cutoff: N.
  Team: +x.x pp to y.y%. Short English causal reason.
  Team: -x.x pp to y.y%. Short English causal reason.
  Other small changes are normalization/path effects from the verified results. T6 warning scan set no fact. T7 was not searched or used; any incidental T7 segment was ignored with no model effect.
  Jede wesentliche Modellbewegung braucht eine eigene Team-Zeile. Diese Struktur wird für nachgelagerte Auswertungen verwendet.
- Exportiere zwingend genau diese vier Hauptlauf-Artefakte:
  1. HTML-Datei in deutscher Sprache: dynamo26_YYYY-MM-DD_de.html
  2. HTML-Datei in englischer Sprache: dynamo26_YYYY-MM-DD_en.html
  3. Prior-State-TXT für nächsten Lauf: dynamo26_prior_state_YYYY-MM-DD_pipeline_v1_0.txt
  4. kurze Changes-TXT: dynamo26_changes_YYYY-MM-DD.txt
- Schreibe die vier Artefakte in das vom Tageslauf bereitgestellte Ausgabeziel. Erzeuge oder beschreibe im Hauptlauf keine lokalen Website-Pfade und keine nachgelagerten Veröffentlichungsartefakte.
- Finale Antwort nur mit allen vier Downloadlinks ausgeben.
- Füge <meta name="format-detection" content="telephone=no"> im head der HTML Dateien ein.
- Füge in die vollständigen Run-HTML-Dateien einen <style>-Block mit dem Standard-CSS aus dynamo26_code_v1_0.txt ein. Fehlendes CSS allein ist kein KO-Kriterium.

Ausgabe:
- Kurze auditierte Chat-Ausgabe.
- HTML-Downloadlink deutsch.
- HTML-Downloadlink englisch.
- Prior-State-TXT-Downloadlink.
- Changes-TXT-Downloadlink.
- Hinweis: Für den nächsten Lauf die Prior-State-TXT verwenden, nicht die HTML.

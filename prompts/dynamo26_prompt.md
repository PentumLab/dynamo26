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
18. BUILD_MATCHUP_MATRIX
19. SIMULATE_PATHS
20. NORMALIZE_PROBABILITIES
21. ATTACH_PROBABILITIES
22. CALCULATE_CRASH_RISK
23. GENERATE_CAUSAL_CHAINS
24. AUDIT_MODEL
25. SELF_CORRECT falls nötig
26. RENDER_OUTPUT oder RENDER_NOT_CURRENT
27. BUILD_CHANGE_SUMMARY
28. EXPORT_ARTIFACTS
29. AUDIT_ARTIFACTS
30. DELIVER

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
- T7 ist vollständig verboten: keine externen Prognosen, Projektionen, Buchmacherquoten, Elo-/Ranking-Prognosen oder Medien-Wahrscheinlichkeiten laden, suchen, öffnen, zitieren oder verwenden. Auch keine Kalibrierung damit.
- Gruppenabschluss-Regel: Jeder Lauf prüft zwei Bedingungen: (1) Ist die Gruppenphase laut offiziellem Turnierkern vorbei? (2) Enthält der Prior-State bereits group_stage_final_qualification_gate_passed=true mit vollständigen Listen qualified_teams, best_third_placed_teams und eliminated_teams? Nur wenn (1)=true und (2)=false/fehlt, muss der Lauf das Gruppenabschluss-/K.o.-Qualifikations-Gate vollständig ausführen. Danach muss der neue Prior-State group_stage_final_qualification_gate_passed=true sowie die finalen Listen speichern. Vor bestandenem Gate dürfen Drittplatzierte nicht endgültig als qualifiziert/ausgeschieden behandelt werden.
- Tabellen-Lebenszyklus ab dem nächsten Tageslauf: Teams mit title_probability 0.0 dürfen nur dann in der Haupt-Wahrscheinlichkeitstabelle in HTML und TITLE_PROBABILITY_PRIOR_TABLE erscheinen, wenn sie im aktuellen Lauf gerade auf 0.0 gefallen bzw. gerade ausgeschieden sind. Teams, die bereits in einem früheren Lauf auf 0.0 gefallen waren, werden nicht weiter als Haupttabellenzeile fortgeführt, müssen aber im Prior-State in eliminated_teams, knockout_eliminated_teams_after_prior_cutoff oder einem Archiv-/Statusabschnitt nachvollziehbar erhalten bleiben. Wenn Restfeld eine positive Titelwahrscheinlichkeit hat, muss die Haupttabelle genau 25 Zeilen enthalten. Freie Tabellenplätze müssen mit noch aktiven Teams gefüllt werden, die fachlich im Restfeld-Aggregat enthalten sind; ausgeschiedene Restfeld-Teams dürfen nicht als Füllteams erscheinen. Die Füllteams sind nach internem Restfeld-Wert absteigend auszuwählen. Wenn diese Werte im Prior-State nicht bestimmt sind, muss der Lauf sie aus Restfeld-Signalen, K.o.-Pfad, Qualifikationsstatus, Gegnerpfad, Teamstärke-/Indexhinweisen und aktuellen Quellen nachvollziehbar ermitteln. Restfeld-Füllteams sind keine reinen Anzeigezeilen: Wenn sie in der Haupttabelle erscheinen, müssen für sie title_probability_percent, final_probability_percent, quarter_final_probability_percent, early_exit_probability_percent und crash_risk_score vollständig modelliert werden. Dasselbe gilt für eine verbleibende Restfeld-Restzeile als Aggregat. Dabei dürfen keine Werte erfunden oder doppelt gezählt werden: Die Summe der Restfeld-Füllteam-Titelwerte plus Restfeld-Restzeile muss exakt der Restfeld-Gesamtwahrscheinlichkeit entsprechen. Die Final-/VF-/Early-Exit-/Crash-Werte müssen aus derselben Pfadsimulation bzw. derselben Restfeld-Komponentenrechnung stammen; wenn das nicht möglich ist, gilt der Lauf als nicht erfolgreich und darf nicht finalisiert werden. Wenn Restfeld-Komponenten neu aus dem Aggregat gehoben werden, muss geprüft werden, ob die Pfadsimulation dadurch andere Teams beeinflusst; nötige Korrekturen an anderen Teams müssen mit Kausalkette dokumentiert werden. Die Restfeld-Restzeile steht immer am Ende der Haupttabelle, solange ein positiver Restfeld-Rest verbleibt. Wenn kein positives Restfeld mehr existiert, wird keine Restfeld-Zeile angezeigt und die Haupttabelle darf weniger als 25 Zeilen haben.
- Summenregeln für Wahrscheinlichkeitstabellen: title_probability_percent muss über die gesamte Haupttabelle inklusive Restfeld-Komponenten und Restfeld-Rest exakt 100.0 ergeben. final_probability_percent muss exakt 200.0 ergeben, weil es zwei Finalplätze gibt. quarter_final_probability_percent muss exakt 800.0 ergeben, weil es acht Viertelfinalplätze gibt. Diese Summenregeln gelten auch dann, wenn ein Restfeld-Rest als Aggregatzeile verbleibt; die Aggregatzeile trägt dann die Summe der in ihr enthaltenen Mikro-Pfade für Title, Final und QF. crash_risk_score ist dagegen ein individueller Risiko-Score und darf nicht aufsummiert werden. early_exit_probability_percent ist ebenfalls kein sinnvoller Summenwert für eine Restfeld-Rest-Aggregatzeile. Für eine Restfeld-Rest-Aggregatzeile müssen crash_risk_score und early_exit_probability_percent leer bleiben, solange sie nicht ausdrücklich als gewichtete Aggregatindikatoren mit eigener Kennzeichnung definiert werden. Wenn die Summenregeln nach Rundung nicht erfüllt sind, muss der Lauf nachjustieren oder als nicht erfolgreich gelten.
- Jede OUTPUT-Aussage braucht eine Kausalkette.
- Die kurze Changes-TXT muss einsprachig englisch und strukturiert sein, im Format:
  DYNAMO26 changes - YYYY-MM-DD
  Status: CURRENT|DEGRADED. New verified matches after prior cutoff: N.
  Team: +x.x pp to y.y%. Short English causal reason.
  Team: -x.x pp to y.y%. Short English causal reason.
  Other small changes are normalization/path effects from the verified results. T6 warning scan set no fact. T7 was not searched, opened or used.
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

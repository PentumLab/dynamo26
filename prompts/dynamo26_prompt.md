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
- Jede OUTPUT-Aussage braucht eine Kausalkette.
- Erzeuge Tageslauf-Artefakte zuerst in website/wm26/staging/YYYY-MM-DD_RUNID.
- Nur wenn Quellen-, Modell- und Artefaktprüfungen bestanden sind, darf veröffentlicht werden:
  python scripts/dynamo26_finalize_run.py --date YYYY-MM-DD --run-id RUNID --status success
- Nur nach erfolgreicher Finalisierung dürfen XPosts erzeugt werden:
  1. Führe XPostPrior.txt als Prompt mit genau diesen Eingaben aus:
     website/wm26/runs/dynamo26_YYYY-MM-DD_en.html
     website/wm26/runs/dynamo26_prior_state_YYYY-MM-DD_pipeline_v1_0.txt
  2. Führe XPostChangeMultiLang.txt als Prompt mit genau diesen Eingaben aus:
     website/wm26/runs/dynamo26_YYYY-MM-DD_en.html
     website/wm26/runs/dynamo26_changes_YYYY-MM-DD.txt
  3. Speichere alle erzeugten XPost-Dateien in website/wm26/xposts.
  4. Aktualisiere danach die XPost-JSON:
     python scripts/dynamo26_update_xposts_json.py --date YYYY-MM-DD
- Nur nach erfolgreicher Finalisierung, erfolgreicher XPost-Erzeugung und bestandener XPost-Pruefung darf der aktuelle Lauf hochgeladen werden:
  powershell -ExecutionPolicy Bypass -File scripts/upload_dynamo26_run.ps1 -Date YYYY-MM-DD
- Vor dem Upload muessen alle XPost-TXT-Dateien und website/wm26/xposts/xposts.json als echtes UTF-8 ohne Unicode-Replacement-Char und ohne literal ?-Ersatzzeichen validiert sein. Bei Verletzung: Upload abbrechen, XPosts/JSON korrekt neu erzeugen, danach erneut pruefen.
- Jede XPost-Sprachversion muss echte UTF-8-Zeichen der Zielsprache verwenden. Deutsch nutzt Umlaute/ß, Franzoesisch und Spanisch nutzen Akzente/Sonderzeichen, Arabisch/Chinesisch/Japanisch nutzen die jeweilige Schrift. ASCII-Umschreibungen wie ae/ue/oe statt Umlauten oder fehlende Akzente sind vor dem Upload zu korrigieren.
- Der Upload gehoert nur zum Erfolgspfad. Bei Fehlerlaeufen, nicht bestandenen Pruefungen oder fehlender XPost-JSON darf nichts hochgeladen werden.
- Wenn der Upload fehlschlaegt: upload_status=FAILED berichten, aber keine fachlichen Artefakte aendern und keinen Fehler-Finalizer nachtraeglich auf bereits erfolgreich finalisierte Artefakte anwenden.
- Bei fehlerhaftem oder nicht bestandenem Lauf dürfen prepared-Dateien und runs_*.json nicht aktualisiert werden. Stattdessen:
  python scripts/dynamo26_finalize_run.py --date YYYY-MM-DD --run-id RUNID --status failed --reason SHORT_REASON
- Exportiere zwingend:
  1. HTML-Datei in deutscher Sprache
  2. HTML-Datei in englischer Sprache
  3. Prior-State-TXT für nächsten Lauf
  4. kurze Changes-TXT
- Finale Antwort nur mit allen vier Downloadlinks ausgeben.
- Füge <meta name="format-detection" content="telephone=no"> im head der HTML Dateien ein.
- Füge in die vollständigen Run-HTML-Dateien einen <style>-Block mit dem Standard-CSS aus dynamo26_code_v1_0.txt ein. Die prepared HTML-Fragmente dürfen diesen Style später entfernen. Fehlendes CSS allein ist kein KO-Kriterium.

Ausgabe:
- Kurze auditierte Chat-Ausgabe.
- HTML-Downloadlink deutsch.
- HTML-Downloadlink englisch.
- Prior-State-TXT-Downloadlink.
- Changes-TXT-Downloadlink.
- Hinweis: Für den nächsten Lauf die Prior-State-TXT verwenden, nicht die HTML.

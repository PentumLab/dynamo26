# DYNAMO26

Dynamo26 first defines a fixed mathematical model for team strength, risk and tournament paths. In other words, it has learned which factors influence winning a football tournament and has built its own model from them.

A Large Language Model (LLM) then evaluates verified match events, tactical control, adaptability, resilience, squad depth, injuries, workload, travel demands and tournament-path difficulty within this framework.

Dynamo26 determines each team’s title probability by independently combining these information signals — without using external forecasts, betting odds or media rankings.

[Description Video](https://dynamo26.pentumlab.com/video.html)

## Usage

1. Upload the model file from `model/`.
2. Upload the latest prior-state TXT file from `state/`.
3. Open `prompts/dynamo26_prompt.md`, copy its contents and paste them into the LLM as the prompt.
4. Run the prompt.

After each completed run, use the newly generated prior-state TXT file for the next run. Do not use the HTML output as prior-state input.

## Independence & Disclaimer

DYNAMO26 is an independent analytical model.

It is not affiliated with or endorsed by FIFA, any football association, team, broadcaster, bookmaker or official data provider.

No betting odds, bookmaker forecasts or external probability rankings are used as model inputs.

The model outputs are analytical estimates and should not be interpreted as guaranteed predictions or betting advice.

## License

Copyright 2026 PentumLab GmbH.

Licensed under the Apache License 2.0. See `LICENSE` and `NOTICE`.

# AIEvalTool Technical Assignment

## Conversational Endpoint
Farmer.Chat - Kenya named Gooey AI a whatsapp based chatbot that gives agricultural information to farmers. This was chosen as my interests and work are primarily in AI and agriculture and Farmer.Chat is among the few that provide AI services to smallholder farmers in India, Kenya, Ethiopia. I also wanted to test how these chatbots would work when a chatbot for Ethiopia is being used by a user in India or elsewhere, and whether appropriate guardrails were set up. Hence, my evaluation used the Kenyan version of Farmer.Chat but the queries had Indian inputs. 

Link for Farmer.Chat: https://www.help.gooey.ai/farmerchat

## Path Chosen and Why?
I started using option A (Evaluate and Report) but realized that the code was not working as expected in my windows device hence I resorted to option B (Critiue and Rebuild) and partially rebuilt some parts of the code (not entirely) as I will detail below.

## 
I downloaded the source file from the provided link i.e., AIEvaltool Version 1.2 (latest version is 2.0 but as per the link provided in the assignment, I used version 1.2)
Link provided: https://github.com/cerai-iitm/AIEvaluationTool/releases/tag/v1.2

The readme.md in the source provides for the initial setup and configuration which largely involved the following:

- Installing software requirements
- Using SqLite (default)
- Creating env variable
- Using Llama3.2:latest model with Ollama (not qwen3.2, as it requires a lot of RAM, not feasible on PCs, I did not have the necessary RAM on my system)
- The below target is set in config files in `src/app/importer/config.json`, `src/app/testcase_executor/config.json` & `src/app/response_analyzer/config.json`

```json
"target": {
        "application_type": "WHATSAPP_WEB",
        "application_name": "Gooey AI",
        "application_url": "https://web.whatsapp.com/",
        "agent_name": "4253105658" # this number is critical as this value is used for searching on whatsapp web
    }
```
Similarly, the db type was set in config files as below:
```json
   {
     "db": {
       "engine_type": "sqlite",
       "file": "AIEvaluationData.db"
     }
   }
```
In some cases, the original tool had key errors, like "db" was written as "databases", such errors can be replaced manually to map in other documents or can be updated using AI agents like co-pilot.

## Test Design Suite
Once, files are installed and configurations made, run ollama, importer script, the interface manager API service and finally the main.py testcase executor with necessary arguments.

### Run test cases
I used the following to run tests:
```python main.py --testplan-id 1  --max-testcases 30 --config "config.json" --domain-strict --execute```

--test-plan-id = 1 indicates Responsible AI evaluation test \
--max-testcases = 30 indicates 30 prompts were sent to the whatsapp based chatbot \
--domain-strict is supposed to ensure agriculture is the only domain on which quesries would be asked (although I am unsure if this parameter actually works as required)

This will start the selenium automation (ensure the Xpaths are apropriate), and the queries would be sent and responses would be stored in the database. I did not add any Datapoints but used the ones already in the DataPoints.json file by default as they had sufficient agriculture examples for a pilot testing.

I used the responsible AI test suite with 30 testcases as farming in India requires highly context specific information and often deals with smallholder farmers where information needs to be carefully shared. 30 testcases is a parsimonious number typically used to justify statistical significance, though in this case statistical significane was not a necessity, 30 seemed like a large enough number that my personal computer could handle in limited time.

## Analysis and code-fix
Once the test cases are run, you have to move to the response analysis folder and run the python code with the RUN name generated automatically during running tests. This will perform the AI evalution based on the selected testplan and strategies in the tool. I faced multiple errors in this, the strategies were not being recognized, upon doing some RCA and a little help from co-pilot and gemini, multiple lines had to be addeed to the ./data/defaults.json file linking the strategy to the necessary models, data source, response types and more, example given below. Once, this was done, the analysis ran fine and as expected.
```
.
.
.
"bias_detection": {
  "model_name": "llama3.2:latest",
  "bias_types": ["gender", "race", "religion", "age", "socioeconomic"]
},
"toxicity": {
  "threshold": 0.5,
  "model_name": "llama3.2:latest"
},
"hallucination": {
  "model_name": "llama3.2:latest",
  "threshold": 0.5
},
.
.
.
```

## Report and PDF output
Finally, once analysis was done the report was generated using the necessary command. The PDf and JSOn can be found in the repo having names
```
AI_Evaluation_Report_Gooey AI_sequence-sparse-magazine-mauris.pdf
AI_Evaluation_Report_Gooey AI_sequence-sparse-magazine-mauris.json
```

## Other trials for dashboard
I got the frontend and backend running for the TDMS dashboard but the application was not working as expected. The testplans, testcases and all other information were not present. This may be because my configuration was not as expected. But I suspect, the code also needs streamlining and is very spreadout at the moment.

## Final analysis of report and data - Conclusions
The current AIEvalTool version 1.2 (not latest, latest is v2.0, I used 1.2 as it was shared for assignment) is not stable and needs heavy editing for proper output.
Even in the final report, many assessments failed as can be seen in the pdf report. The report itself needs to be updated, currently it only shows plan, metric name, score and summary, there is no way I could tell what the response was to a query or what queries were sent to the whatsapp chatbot making anaysis challenging for a human. I am aware that some of these issues may be as the tool was developed with qwen3.2 in mind, but many organizations or even lay users might not be able to run such huge models on the field or in constrained devices. Hence, care must be taken that the limitations are explicitly stated and if possible the tool be made such that it can run using smaller language models. 

The prompts used for the AI evalution is currently obscured. While this may be useful and desirable for a user who is not technically adept, an option should be provided such that the prompt, query and response is clearly listed in the final or intermediary outputs in the tool. In the absence of such a capability, it is challenging to judge how good of bad these tool actually performs.

The TDMS/UI based tool did not work as expected and the linkages were not functioning and needs a complete fix.

I also purposefuly opted for the whatsapp based path as the selenium automation needs careful screening. Whatsapp chatbots send multiple responses to one query. Its unclear whether the tool as it is now, takes all the output from a single query to to the assessment or only a single or perhaps the first few. How does it handle media like audio/video etc is also not clear.
---
aliases:
- "Amazon Translate"
---

# Amazon Translate

- Is a text translation service based in ML
- Translates text from native language to other languages one word at a time
- Translation process has two parts:
    - Encoder reads the source text => [[cloudformation|outputs]] a semantic representation (meaning)
    - Decoder reads in the meaning => writes to the target language
- Attention mechanism ensures "meaning" is translated
- [[AWS_SA_PRO_Obsidian_Notes/Master/16-other/textract|textract]] is capable to detect the source text language
- Use cases:
    - Multilingual user experience
    - [[AWS_SA_PRO_Obsidian_Notes/Master/Machine_Learning/Translate|Translate]] incoming data (social media/news/communications)
    - Language-independence for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]]: we might have other services such as [[AWS_SA_PRO_Obsidian_Notes/Master/16-other/comprehend|comprehend]], [[AWS_SA_PRO_Obsidian_Notes/Master/16-other/transcribe|transcribe]] and [[AWS_SA_PRO_Obsidian_Notes/Master/16-other/polly|polly]] which operate on information; [[AWS_SA_PRO_Obsidian_Notes/Master/16-other/transcribe|transcribe]] will make this services operate in a language independent way. It can used to analyze data stored in [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]], [[rds]], DDB, etc.
    - Commonly used for integration with other services/Apps/platforms
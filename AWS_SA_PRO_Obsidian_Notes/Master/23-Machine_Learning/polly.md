---
aliases:
- "Amazon Polly"
---

# Amazon Polly

- Converts text in "life-like" speech
- The products takes text in specific languages and [[cloudformation|outputs]] speech in that specific language. [[AWS_SA_PRO_Obsidian_Notes/Master/Machine_Learning/Polly|Polly]] does not do translation!
- There 2 modes that [[AWS_SA_PRO_Obsidian_Notes/Master/Machine_Learning/Polly|Polly]] operates in:
    - Standard TTS:
        - Uses a concatenative architecture
        - Takes phonemes (smallest units of sound) to build patterns of speech
    - Neural TTS:
        - Takes phonemes, generate spectograms, it puts those spectograms through a vocoder form which gets the output audio
        - Much advanced way of generating human-like speech
- Output formats: MP3, Ogg Vorbis, PCM
- [[AWS_SA_PRO_Obsidian_Notes/Master/Machine_Learning/Polly|Polly]] is capable of using the Speech Synthesis Markup Language (SSML). This is a way we can provide additional control over how [[AWS_SA_PRO_Obsidian_Notes/Master/Machine_Learning/Polly|Polly]] generates speech. We can get [[AWS_SA_PRO_Obsidian_Notes/Master/Machine_Learning/Polly|Polly]] to emphasis certain part of the text or do certain pronunciation (whispering, Newscaster speaking style)
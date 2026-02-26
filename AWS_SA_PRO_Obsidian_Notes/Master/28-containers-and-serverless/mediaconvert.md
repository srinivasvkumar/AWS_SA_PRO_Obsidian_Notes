---
aliases:
- "Choosing between ET and MC"
- "Elastic Transcoder and AWS Elemental MediaConvert"
---

# Elastic Transcoder and AWS Elemental MediaConvert

- MediaConvert a fairly new AWS product replacing Elastic Transcoder
- MediaConvert is a superset of features provided by Transcoder
- Both of the systems are file based video transcoding systems
- They are serverless transcoding services for which we pay per use
- For both we add jobs to pipelines (ET) or queues (MC)
- Files are loaded from [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]], processed and stored back to [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]]
- MC supports [[eventbridge]] for job signalling
- ET/MC architecture example:
    [ET/MC architecture](AWS_SA_PRO_Obsidian_Notes/Master/09-containers-and-serverless/images/ElasticTranscoder&MediaConvert.png)
- We use these products when we need to the media convert in a serverless, event-driven media processing pipeline

## Choosing between ET and MC

- ET is legacy, by default we should chose MC
- MC supports more codecs, design for larger volume and parallel processing
- MC supports reserved pricing
- ~~ET required for WebM(VP8/VP9), animated GIF, MP3, Vorbis and WAV.~~ All of these codecs are supported for MC. For everything else we should use MediaConvert
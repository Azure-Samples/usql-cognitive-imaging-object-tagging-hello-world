---
services: data-lake-analytics
platforms: dotnet
author: saveenr
---

# USQL/Cognitive Imaging Hello World

## Imaging  scenarios
### Image Tagging

```
REFERENCE ASSEMBLY ImageCommon;
REFERENCE ASSEMBLY FaceSdk;
REFERENCE ASSEMBLY ImageEmotion;
REFERENCE ASSEMBLY ImageTagging;
REFERENCE ASSEMBLY ImageOcr;

@imgs =
    EXTRACT FileName string, ImgData byte[]
    FROM @"/images/{FileName:*}.jpg"
    USING new Cognition.Vision.ImageExtractor();

// Extract the number of objects on each image and tag them 
@objects =
    PROCESS @imgs 
    PRODUCE FileName,
            NumObjects int,
            Tags string
    READONLY FileName
    USING new Cognition.Vision.ImageTagger();
```

### Extract emotions from human faces

```
@emotions =
    PROCESS @imgs
    PRODUCE FileName string,
            NumFaces int,
            Emotion string
    READONLY FileName
    USING new Cognition.Vision.EmotionAnalyzer();
```

### Estimate age and gender for human faces


```
@ocrs =
        PROCESS @imgs
        PRODUCE FileName,
                Text string
        READONLY FileName
        USING new Cognition.Vision.OcrExtractor();
```


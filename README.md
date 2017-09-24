---
services: data-lake-analytics
platforms: dotnet
author: saveenr
---

# USQL/Cognitive Imaging Hello World

## Load the assemblies

```
REFERENCE ASSEMBLY ImageCommon;
REFERENCE ASSEMBLY FaceSdk;
REFERENCE ASSEMBLY ImageEmotion;
REFERENCE ASSEMBLY ImageTagging;
REFERENCE ASSEMBLY ImageOcr;
```

## Get the image data

```
@imgs =
    EXTRACT 
        FileName string, 
        ImgData byte[]
    FROM @"/usqlext/samples/cognition/{FileName}.jpg"
    USING new Cognition.Vision.ImageExtractor();
```

## Extract the number of objects on each image and tag them 

```
@tags =
    PROCESS @imgs 
    PRODUCE FileName,
            NumObjects int,
            Tags SQL.MAP<string, float?>
    READONLY FileName
    USING new Cognition.Vision.ImageTagger();
   
```

## Write out the tagging information

```
@tags_serialized =
    SELECT FileName,
           NumObjects,
           String.Join(";", Tags.Select(x => String.Format("{0}:{1}", x.Key, x.Value))) AS TagsString
    FROM @tags;

OUTPUT @tags_serialized
    TO "/tags.csv"
    USING Outputers.Csv();
```

## Extract Emotion from human faces

```
@emotions_from_extractor =
    EXTRACT FileName string, 
        NumFaces int, 
        FaceIndex int, 
        RectX float, 
        RectY float, 
        Width float, 
        Height float, 
        Emotion string, 
        Confidence float
    FROM @"/usqlext/samples/cognition/{FileName}.jpg"
    USING new Cognition.Vision.EmotionExtractor();
```

### Estimate age and gender for human faces

```
@faces_from_extractor =
    EXTRACT FileName string, 
        NumFaces int, 
        FaceIndex int, 
        RectX float, RectY float, Width float, Height float, 
        FaceAge int, 
        FaceGender string
    FROM @"/usqlext/samples/cognition/{FileName}.jpg"
    USING new Cognition.Vision.FaceDetectionExtractor();
```

## Detect Text in Images

```
@ocrs =
    PROCESS @imgs
    PRODUCE FileName,
            Text string
    READONLY FileName
    USING new Cognition.Vision.OcrExtractor();
```


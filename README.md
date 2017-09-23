# USQL/Cognitive Hello World

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

## Text scenarios


### Input data

```
REFERENCE ASSEMBLY [TextCommon];
REFERENCE ASSEMBLY [TextSentiment];
REFERENCE ASSEMBLY [TextKeyPhrase];

@WarAndPeace =
    EXTRACT No int,
            Year string,
            Book string,
            Chapter string,
            Text string
    FROM @"/usqlext/samples/cognition/war_and_peace.csv"
    USING Extractors.Csv();
```

### Extract key phrases for each paragraph

```
@keyphrase =
    PROCESS @WarAndPeace
    PRODUCE No,
            Year,
            Book,
            Chapter,
            Text,
            KeyPhrase string
    READONLY No,
            Year,
            Book,
            Chapter,
            Text
    USING new Cognition.Text.KeyPhraseExtractor();

// Tokenize the key phrases.
@kpsplits =
    SELECT No,
        Year,
        Book,
        Chapter,
        Text,
        T.KeyPhrase
    FROM @keyphrase
        CROSS APPLY
            new Cognition.Text.Splitter("KeyPhrase") AS T(KeyPhrase);
```


### Perform sentiment analysis on each paragraph

```
@sentiment =
    PROCESS @WarAndPeace
    PRODUCE No,
            Year,
            Book,
            Chapter,
            Text,
            Sentiment string,
            Conf double
    READONLY No,
            Year,
            Book,
            Chapter,
            Text
    USING new Cognition.Text.SentimentAnalyzer(true);
```

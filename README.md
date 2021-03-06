# BagOfWords_SentimentAnalysis_Template

# OverView

In this ML Engine we have implemented Bag Of words model using Spark - MlLib - 1.5.1, predictionIO 0.10.0-incubating-rc1 and scala 2.10.5. Bag Of Words model reffer to [Thumbs up? Sentiment Classification using Machine Learning
Techniques] (http://www.cs.cornell.edu/home/llee/papers/sentiment.pdf )

In this engine, Data preprocessing part include tokenization, negation, term frequency and both unigram + bigram as features, which is trained in Algorithm part of DASE model with NaiveBayes classifier. Training model can be customized with other calssifiers.

## Usage

### Event Data Requirements
By default, the template requires the following events to be collected (/data/import_eventserver.py ):

- user train event, to train model with specific dataset

### Input Query
```
{"phrase": "It is good"}
```

### Output Predicted Result
- it returns sentiment of phrase with 1.0 - positive or 0.0 - negative with relative probabilistic score.
```
{"Sentiment":1.0,"Score":0.92}
```

### Dataset
We will be using a twitter sentiment analysis data set from [Twitter Sentiment Analysis Trainig Corpus] (http://thinknook.com/twitter-sentiment-analysis-training-corpus-dataset-2012-09-22/)

Trainig data sample :

```
ItemID,Sentiment,SentimentSource,SentimentText
1,0,Sentiment140, is so sad for my APL friend.............
2,0,Sentiment140, I missed the New Moon trailer...
3,1,Sentiment140, omg its already 7:30 :O

```

The training sample events have the following format (Generated by data/import_eventserver.py):

```
client.create_event(
      event="train",
      entity_type="phrase",
      entity_id=count,
      properties= { "phrase" : data[3], "sentiment": float(data[1]) }
    )
```

## Install and Run PredictionIO
Install PredictinIO from [Apache PredictionIO](http://predictionio.incubator.apache.org/install/).
Let's say you have installed PredictionIO at /home/yourname/PredictionIO/. For convenience, add PredictionIO's binary command path to your PATH, i.e. /home/yourname/PredictionIO/bin
```
$ PATH=$PATH:/home/yourname/PredictionIO/bin; export PATH
```
Once you have completed the installation process, please make sure all the components (PredictionIO Event Server, Elasticsearch, and HBase) are up and running.

```
$ pio-start-all
```
You can check the status by running:

```
$ pio status
```
## Download Template
To get template clone the below repository by executing the following command in the directory where you want the code to reside:

```
git clone https://github.com/peoplehum/BagOfWords_SentimentAnalysis_Template
```

## Generate an App ID and Access Key
Let's assume you want to use this engine in an application named "testApp". You will need to collect some training data for machine learning modeling. You can generate an App ID and Access Key that represent "testApp" on the Event Server easily:
```
$ pio app new testApp
```
You should find the following in the console output:
```
...
[INFO] [App$] Initialized Event Store for this app ID: 1.
[INFO] [App$] Created new app:
[INFO] [App$]       Name: testApp
[INFO] [App$]         ID: 1
[INFO] [App$] Access Key: 3mZWDzci2D5YsqAnqNnXH9SB6Rg3dsTBs8iHkK6X2i54IQsIZI1eEeQQyMfs7b3F
```
Take note of the Access Key and App ID. You will need the Access Key to refer to "testApp" when you collect data. At the same time, you will use App ID to refer to "testApp" in engine code.

$ pio app list will return a list of names and IDs of apps created in the Event Server.

```
$ pio app list
[INFO] [App$]                 Name |   ID |                                                       Access Key | Allowed Event(s)
[INFO] [App$]               testApp |    1 | 3mZWDzci2D5YsqAnqNnXH9SB6Rg3dsTBs8iHkK6X2i54IQsIZI1eEeQQyMfs7b3F | (all)
[INFO] [App$]               MyApp |    2 | io5lz6Eg4m3Xe4JZTBFE13GMAf1dhFl6ZteuJfrO84XpdOz9wRCrDU44EUaYuXq5 | (all)
[INFO] [App$] Finished listing 2 app(s).
```

To use template with above created application, modify appName in engine.json
```
"datasource": {
    "params" : {
      "appName" : "testApp",
      "evalK" : 3
    }
  }
```



## Collecting Data

Next, let's collect some training data. By default, the Engine Template reads 2 properties of a user record: SentimentText(phrase) and sentiment.

You can send these data to PredictionIO Event Server in real-time easily by making a HTTP request or through the EventClient of an SDK.

A Python import script import_eventserver.py is provided in the template to import the data to Event Server using Python SDK.
Replace the value of access_key parameter by your applications's Access Key and run:

```python
$ pip install predictionio
$ cd BagOfWords_SentimentAnalysis_Template
$ python data/import_eventserver.py --access_key 3mZWDzci2D5YsqAnqNnXH9SB6Rg3dsTBs8iHkK6X2i54IQsIZI1eEeQQyMfs7b3F --file data/train.csv
```
You should see the following output:
```
Importing data...
1578627 events are imported.
```
This python script converts the data file to proper events formats as needed by the event server.
Now the training data is stored as events inside the Event Store.

## Deploy the Engine as a Service
Now you can build, train, and deploy the engine. First, make sure you are under the Template directory.

### Build

Start with building your Sentimant Analysis engine.
```
$ pio build
```
This command should take few minutes for the first time; all subsequent builds should be less than a minute. You can also run it with --verbose to see all log messages.

Upon successful build, you should see a console message similar to the following.
```
[INFO] [Console$] Your engine is ready for training.
```

### Training the Predictive Model

Train your engine.

```
$ pio train
```

In case of very large dataset allocate more memory
```
$ SPARK_MEM="6g" pio train
```
When your engine is trained successfully, you should see a console message similar to the following.

```
[INFO] [CoreWorkflow$] Training completed successfully.
```
### Deploying the Engine

Now your engine is ready to deploy.

```
$ pio deploy
```
This will deploy an engine that binds to http://localhost:8000. You can visit that page in your web browser to check its status.

You can specify port where to deploy
```
$ pio deploy --port 8088
```
### Execute Query
Run below request for processing query on serving layer, it will return sentiment and its probabilistic score. 
```
curl -k -H "Content-Type: application/json" -d '{"phrase": "Freinds TV series is not good"}' https://localhost:8000/queries.json
```
# Future Work
As it follows bag of words model, it does not preserve order of words or sentiment of words. It is upon frequecy of words and ngrams model, that can be further improved for better accuracy for ex. by adding part of speech tagging, or any other feature extraction techniques. 


# Relative Issues
For any problem, you can create issue here and for merging new changes make pull request. For any further query you can communicate on bansari.jan93@gmail.com

## License

This algorithm is under [Apache 2
license](http://www.apache.org/licenses/LICENSE-2.0.html).

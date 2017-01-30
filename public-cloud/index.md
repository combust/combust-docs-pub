# Public Cloud

Take the Combust Cloud for a spin in 3 easy steps:

### Step 1: Register

Register for Combust Cloud at combust.ml. Once registered, you will receive your `container name` and `platform token` - you'll need both to deploy your ML pipeline as a service.


### Step 2: Train a Model

Train your own ML Pipeline or use the [MLeap AirBnb demo](https://github.com/combust/mleap-demo/blob/master/notebooks/airbnb-price-regression.ipynb) to train either a linear or a random forrest regression.

#### Spark/PySpark

If you are training your own Spark or PySpark pipeline, refer to the AirBnb demo on how to create a `SparkBundleContext` and serialize your pipeline to an MLeap Bundle. After fitting your pipeline, you will have to:

```scala
// First create a Spark Bundle Context
val sbc = SparkBundleContext()

// Second serialize your pipeline (sparkPipelineLr) to an MLeap Bundle
for(bf <- managed(BundleFile("jar:file:/tmp/my.model.lr.zip"))) {
        sparkPipelineLr.writeBundle.save(bf)(sbc).get
  }
  
// Third, .deploy() to your private sandbox
{     
    implicit val context = sbc
    Await.result(sparkPipelineLr.deploy("http://models.combust.ml:65327", "my_username", "my_model_lr", platform_token), 10.seconds)
}
```


#### Scikit-Learn

You can train your own Scikit-Learn Pipeline or refer to the python version of the AirBnb demo. After fitting your ML Pipeline, all you have to do is call `.deploy()`:

```python
# model_pipeline is a Pipeline made up of Feature Transformers and an Estimator
model_pipeline.fit(X_train, y_train)

// .deploy() to your private sandbox
model_pipeline.deploy("http://models.combust.ml:65327", "my_username", "my_model_lr", platform_token)
```

### Step 3: Query Your Service

The first thing you'll need is a LeapFrame, which contains the schema and your data. If you are following the AirBnb demo, you can get the [LeapFrame](https://s3-us-west-2.amazonaws.com/mleap-demo/frame.json) from our S3 bucket.
We also have client libraries available - read more about them [here](clients/index).

Here is what a LeapFrame looks like:

```json
{
  "schema": {
    "fields": [{
      "name": "state",
      "type": "string"
    }, {
      "name": "bathrooms",
      "type": "double"
    }, {
      "name": "square_feet",
      "type": "double"
    }, {
      "name": "bedrooms",
      "type": "double"
    }, {
      "name": "security_deposit",
      "type": "double"
    }, {
      "name": "cleaning_fee",
      "type": "double"
    }, {
      "name": "extra_people",
      "type": "double"
    }, {
      "name": "number_of_reviews",
      "type": "double"
    }, {
      "name": "review_scores_rating",
      "type": "double"
    }, {
      "name": "room_type",
      "type": "string"
    }, {
      "name": "host_is_superhost",
      "type": "string"
    }, {
      "name": "cancellation_policy",
      "type": "string"
    }, {
      "name": "instant_bookable",
      "type": "string"
    }]
  },
  "rows": [["NY", 2.0, 1250.0, 3.0, 50.0, 30.0, 2.0, 56.0, 90.0, "Entire home/apt", "1.0", "strict", "1.0"]]
}
```

Then query using either curl, scala or python:

cURL:

```bash
curl
```

scala
```scala
// Initialize the client
val client = ml.combust.model.client.Client("http://localhost:65327")

// Generate the LeapFrame
val s = scala.io.Source.fromURL("https://s3-us-west-2.amazonaws.com/mleap-demo/frame.json").mkString
val bytes = s.getBytes("UTF-8")
var leapFrame = FrameReader("ml.combust.mleap.json").fromBytes(bytes).get

// Query your service
val result = Await.result(client.transform(TransformRequest().withUsername("my_username").withModelId("my_model_lr").withFrame(leapFrame)), 10.seconds)
val leapFrameFromServer = result.frame
```


```python

```

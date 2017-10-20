# [PredictionIO](https://predictionio.incubator.apache.org) regression engine for [Heroku](http://www.heroku.com) 

A machine learning app deployable to Heroku with the [PredictionIO buildpack](https://github.com/heroku/predictionio-buildpack). Use [Spark's Linear Regression algorithm](https://spark.apache.org/docs/1.6.3/mllib-linear-methods.html#regression) to predict a label from a vector of values.


## Demo Story 🐸

This engine demonstrates prediction of a student's **class grade** based on their grade on an **aptitude test**. The model is trained with a small, example data set.

Five **students'** aptitude and class grades are provided in the [included training data](data/).


## How To 📚

✏️ Throughout this document, code terms that start with `$` represent a value (shell variable) that should be replaced with a customized value, e.g `$EVENTSERVER_NAME`, `$ENGINE_NAME`, `$POSTGRES_ADDON_ID`…

### Deploy to Heroku

Please follow steps in order.

1. [Requirements](#user-content-requirements)
1. [Regression engine](#user-content-regression-engine)
   1. [Create the engine](#user-content-create-the-engine)
   1. [Import data](#user-content-import-data)
   1. [Deploy the engine](#user-content-deploy-the-engine)
   1. [Scale-up](#user-content-scale-up)
   1. [Retry release](#user-content-retry-release)
   1. [Evaluation](#user-content-evaluation)

### Usage

Once deployed, how to work with the engine.

* 🎯 [Query for predictions](#user-content-query-for-predictions)
* [Diagnostics](#user-content-diagnostics)
* [Local development](#user-content-local-development)


# Deploy to Heroku 🚀

## Requirements

* [Heroku account](https://signup.heroku.com)
* [Heroku CLI](https://toolbelt.heroku.com), command-line tools
* [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)

## Regression Engine

### Create the engine

```bash
git clone \
  https://github.com/heroku/predictionio-engine-regression.git \
  pio-engine-regress

cd pio-engine-regress

heroku create $ENGINE_NAME --buildpack https://github.com/heroku/predictionio-buildpack.git
```

### Import data

Initial training data is automatically imported from [`data/initial-events.json`](data/initial-events.json).

👓 When you're ready to begin working with your own data, see [data import methods in CUSTOM docs](https://github.com/heroku/predictionio-buildpack/blob/master/CUSTOM.md#user-content-import-data).

### Deploy the engine

```bash
git push heroku master

# Follow the logs to see training & web start-up
#
heroku logs -t
```

⚠️ **Initial deploy will probably fail due to memory constraints.** Proceed to scale up.

## Scale up

Once deployed, scale up the processes and config Spark to avoid memory issues. These are paid, [professional dyno types](https://devcenter.heroku.com/articles/dyno-types#available-dyno-types):

```bash
heroku ps:scale \
  web=1:Standard-2X \
  release=0:Performance-L \
  train=0:Performance-L
```

## Retry release

When the release (`pio train`) fails due to memory constraints or other transient error, you may use the Heroku CLI [releases:retry plugin](https://github.com/heroku/heroku-releases-retry) to rerun the release without pushing a new deployment:

```bash
# First time, install it.
heroku plugins:install heroku-releases-retry

# Re-run the release & watch the logs
heroku releases:retry
heroku logs -t
```

## Evaluation

This engine is already setup for PredictionIO's [hyperparamter tuning](https://predictionio.incubator.apache.org/evaluation/paramtuning/).

```bash
heroku run bash --size Performance-L
$ cd pio-engine/
$ pio eval \
    org.template.regression.MeanSquaredErrorEvaluation \
    org.template.regression.EngineParamsList \
    -- $PIO_SPARK_OPTS
```

✏️ Memory parameters are set to fit the [dyno `--size`](https://devcenter.heroku.com/articles/dyno-types#available-dyno-types) set in the `heroku run` command.

# Usage ⌨️

## Query for predictions

The linear regression model should only be queried with values within the training range, `60` to `95` for the sample data.

```bash
curl -X "POST" "http://$ENGINE_NAME.herokuapp.com/queries.json" \
     -H "Content-Type: application/json; charset=utf-8" \
     -d $'{"vector": [ 80 ]}'
```

Sample Response:

```json
{"label": 78.32954840770623}
```

## Diagnostics

If you hit any snags with the engine serving queries, check the logs:

```bash
heroku logs -t --app $ENGINE_NAME
```

If errors are occuring, sometimes a restart will help:

```bash
heroku restart --app $ENGINE_NAME
```

## Local Development

If you want to customize an engine, then you'll need to get it running locally on your computer.

➡️ **Setup [local development](https://github.com/heroku/predictionio-buildpack/blob/master/DEV.md)**

### Import sample data

```bash
bin/pio app new regress
PIO_EVENTSERVER_APP_NAME=regress data/import-events -f data/initial-events.json
```

### Run `pio`

```bash
bin/pio build
bin/pio train
bin/pio deploy
```

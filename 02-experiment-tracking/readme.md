**Experiment tracking**
---


# Intro

## Important concepts
* ML experiement

## Whats's experiment tracking

Is the porcess of keeping track of all the **relevant information** from an ML experiment, which includes:

* Surce code
* Enviroment
* Data
* Mode
* Hyperparameters
* Metrics

## Why is experiment tracking so important?

Three main reasons:
* Reproducibility
* Organization
* Optimization

## Why is not enough tracking experiments in spreadsheets?
* Error prone: fill manually the metrics
* No standard format
* Visibility and collaboration

![Example of spreedsheet](./images/spreedsheet.png)

---
## **MLflow**

Is a tool specially created to tracking

Definition: *'An open source platform for the machine learnig lifecycle'*

In practice, it's just a Python package that can be installed with pip, and it contains 4 main modules:
* Tracking
* Models
* Model registry
* Projects

### Tracking experiments with MLflow
The module Tracking of MLflow allows us to organize our experiments into runs, and keep track of:

* Parameters: hyperparameters, path of the dataset, preprosessing
* Metrics: any evaluation metric, accuracy, f1, etc.
* Metadata: example: tags
* Artifacts: example: a visualization
* Models:

Along with this information, MLflow automatically logs extrainformation about the run:
* Source code
* Version of the code (git commit)
* Start and end time
* Author

### Demo

These are the usage commands:
![MLflow Usage](./images/mlflow_usage.png)


# Getting stared with MLflow

## Install package
We need to install the package, the video show the next:

* mlflow
* jupyter
* scikit-learn
* pandas 
* seaborn
* hyperopt
* xgboost

I use conda enviroment, I had to use conda-forge to install mlflow and hyperopt.

## Interface

We can open the user interface with the command `mlflow ui`, but this way we can't go to the model module. To fix this, we user the command: `mlflow ui --backen
d-store-uri sqlite:///mlflow.db`

So, we'll see this:
![Main screen of mlflow ui](./images/mlflow_ui.png)

## Adding MLflow to a existing notebook

Now, we going to use the notebook trained in the session one [duration-prediction.ipynb](duration-prediction.ipynb), and we'll add tracking lines.

* Firstable we need to import the mlflow library
* Set the tracking uri that we defined when we ran mlflow ui (`sqlite:///mlflow.db`)
* Set the experiment, in this example the name is `nyc-taxi-experiment`

This way:

```
import mlflow

mlflow.set_tracking_uri('sqlite:///mlflow.db')
mlflow.set_experiment('nyc-taxi-experiment')
```

To record the performance of the model, we need to star a new run
, next we can add tags like the developer, and next we can add params like which data is used and the params of the model, also we log the metric of the model: 

```
with mlflow.start_run():

    mlflow.set_tag('developer','jaimeh94')

    mlflow.log_param('train-data-path','./data/green_tripdata_2021-01.parquet')
    mlflow.log_param('valid-data-path','./data/green_tripdata_2021-02.parquet')
    
    alpha = 0.01
    mlflow.log_param('alpha',alpha)
    lr = Lasso(alpha)
    lr.fit(X_train, y_train)

    y_pred = lr.predict(X_val)

    rmse = mean_squared_error(y_val, y_pred, squared=False)

    mlflow.log_metric('rmse',rmse)
```

We can see the first run in the mlflow ui:

![first mlflow run](./images/mlflow_run.png)

# Experiment tracking with MLflow

We going to see how to add parameter tuningto the notebook, how it looks in MLflow, select the one, and autolog to make log easier.

For this we going to do a test with the xgboost library, also we'll use the hyperopt library (*Distributed Asynchronous Hyper-parameter Optimization*)

We define the range of the parameters to iterate with hyperopt:

```
search_space = {
    'max_depth': scope.int(hp.quniform('max_depth', 4, 100, 1)),
    'learning_rate': hp.loguniform('learning_rate', -3, 0),
    'reg_alpha': hp.loguniform('reg_alpha', -5, -1),
    'reg_lambda': hp.loguniform('reg_lambda', -6, -1),
    'min_child_weight': hp.loguniform('min_child_weight', -1, 3),
    'objective': 'reg:linear',
    'seed': 42
}
```

The model og ML with tracking into a function:

```
def objective(params):
    with mlflow.start_run():
        mlflow.set_tag("model", "xgboost")
        mlflow.log_params(params)
        booster = xgb.train(
            params=params,
            dtrain=train,
            num_boost_round=1000,
            evals=[(valid, 'validation')],
            early_stopping_rounds=50
        )
        y_pred = booster.predict(valid)
        rmse = mean_squared_error(y_val, y_pred, squared=False)
        mlflow.log_metric("rmse", rmse)

    return {'loss': rmse, 'status': STATUS_OK}
```

And we excecute to do the iterations:

```
best_result = fmin(
    fn=objective,
    space=search_space,
    algo=tpe.suggest,
    max_evals=50,
    trials=Trials()
)
```

Now in the MLflow ui we have many runs:

![Runs](./images/mlflow_runs_xgboost.png)

Here we can compare it in the interface nad we see three  diferent graphics:

* Parallel coodinates plot:

![parallel](./images/mlflow_parallel.png)

* Scatter plot

![scatter](./images/mlflow_scatter.png)

* Contour plot

![contour](./images/mlflow_contour.png)

## Select the best model
To select the best model we just can sort by rmse in the mlflow interface.

![sorted](./images/mlflow_sorted.png)

Clicking in this we can see the parameters of that model.

![best par](./images/best_params.png)

Now we'll this parameters to train a model and see autolog

## Autolog

First we need to call it `mlflow.xgboost.autolog()`

We get more params

![auto params](./images/auto_params.png)

![artifacst](./images/auto_artifacts.png)

# Model management

Machine Learnign Lifecycle

![mlc](./images/mlc.png)

to save our models with MLflow we need to use this line at the end of the run `mlflow.log_artifact(local_path='models/lin_reg.bin', artifact_path='models_pickle)`

## Load models with mlflow

We can load as we see in the ui:
![load](./images/load.png)

![log mod](./images/logg_models.png)


Save the models with the mlflow format is very useful because is a standard
![mlflow model](./images/mlflow_model_format.png)

# Model Registry

The model registry component is a centralized model store, set of APIs, and a UI, to collaboratively manage the full lifecycle of an MLflow Model.

It provides:
* Model lineage
* Model versioning
* stage transitions, and
* Annotations

To promote a model we look the time in training, the rmse and the model size in MB
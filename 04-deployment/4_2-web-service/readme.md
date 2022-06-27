## Deploying a model as a web-service

* Creating a virtual environment with Pipenv
* Creating a script for predictiong
* Putting the script into a Flask app
Running with flask
```
python predict.py
```

Running with gunicorn
```
gunicorn --bind=0.0.0.0:9696 predict:app
```

* Packaging the app to Docker

Build image
```bash
docker build -t ride-duration-prediction-service:v1 .
```

Run image
```bash
docker run -it --rm -p 9696:9696  ride-duration-prediction-service:v1
```
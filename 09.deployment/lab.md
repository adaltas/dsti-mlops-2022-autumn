## 5. Deploy a model

We will follow [this tutorial](https://www.freecodecamp.org/news/end-to-end-machine-learning-project-turorial/).

### 5.1 Prepare and test the app locally

**Objectives:**

- See the steps of a model deployment.

**Dependencies:**

- script: `model.py` in zip file in `lab-resources`

**Instructions:**

By now, we created and tested a model. But we only have a source code, which is not enough to deploy. First we need to save the model we want to deploy as a .bin file. Add the following code to the end of the `model.py`. Run it to produce the pickled model.

``` python
import pickle
##dump the model into a file
with open("model.bin", 'wb') as f_out:
    pickle.dump(lr, f_out) # write final_model in .bin file
    f_out.close()  # close the file
```

Served model will predict wine quality, but we don't have the prediction function yet. Here is the function we will use.

``` python
import pandas as pd

def predict_wine_quality(data, model):
    
    if type(data) == dict:
        df = pd.DataFrame(data)
    else:
        df = data
    
    #y_pred = model.predict(df)
    y_pred = model.predict(df).round().astype(int)
    return y_pred
```

1. Create new directory `mkdir wine_app`
2. Deactivate current virtual environment and create a new one: `conda create --name env_wine_app python=3.9` and activate it 
3. Copy `requirements.txt` from the previous lab (to `wine_app` directory) and add `flask` and `gunicorn`.
4. Install the requirements: `pip install -r requirements.txt`
4. Create `main.py`:

```python
from flask import Flask

##creating a flask app and naming it "app"
app = Flask('app')

@app.route('/test', methods=['GET'])
def test():
    return 'Pinging Model Application!!'

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0', port=9696)
```

5. Run it in the terminal (be careful of folders and virt. env): `python main.py`
6. Open the link written in the terminal and add `/test`
7. You can leave this running and open new terminal to continue the lab, or close it. Don't forget to activate the virtual environment, whenever you open new terminal window: `conda activate env_wine_app`.  
8. In `wine_app` directory, create sub-directory `model_files`: `mkdir model_files`
9. Inside `model_files` create `ml_model.py` and copy inside the `predict_wine_quality`function (above) 
10. Copy pickled model `model.bin` to `model_files`
11. In `model_files`, create empty `__init__.py` file to indicate that the directory is a package.
12. Go back to the `main.py` and add the following code:

Imports:
```python
import pickle
from flask import Flask, request, jsonify
from model_files.ml_model import predict_wine_quality
```

Copy `predict` function and it's route (before the line `if __name__ == '__main__'`) :

```python
@app.route('/predict', methods=['POST'])
def predict():
    wine_sample = request.get_json()
    print(wine_sample)
    with open('./model_files/model.bin', 'rb') as f_in:
        model = pickle.load(f_in)
        f_in.close()
    predictions = predict_wine_quality(wine_sample, model)

    result = list(predictions)
    
    # To be able to serialize it (int64 not serialisable)
    formatted_result = []
    for el in result:
        formatted_result.append(int(el))
        
    return jsonify(formatted_result)
```

12. Save and go to the terminal. Close the terminal window running from before (from the `main.py /test` test). If needed, reactivate the `env_wine_app` and go to the  `wine_app` directory. There, run `main.py`.
13. Open new text file (doesn't matter where) and copy inside (`test_app_locally.py`): 

```python
new_data = {
    'fixed acidity': [1.2, 2.3, 1.3],
    'volatile acidity': [0.1, 0.8, 0.7],
    'citric acid': [0.5, 1.7, 1.1],
    'residual sugar': [3.0, 13.0, 7.6],
    'chlorides': [2.0, 15.0, 13.4],
    'free sulfur dioxide': [10.0, 72.7, 75.0],
    'total sulfur dioxide': [33.0, 135.0, 67.0],
    'density': [0.99, 1.007, 1.0],
    'pH': [3.0, 9.0, 6.7],
    'sulphates': [0.3, 1.8, 0.9],
    'alcohol': [9.3, 13.4, 12.2]
    }

import requests

url = "http://localhost:9696/predict"
r = requests.post(url, json = new_data)
r.text.strip()
print(r.text.strip())
```

When you run it, it should return a list with predictions. If you get an error saying thet the module `requests` was not found, `pip install requests` and run it again.

**NOTE:** If you didn't run the `main.py` before and left it running, this will not work!

### 5.2 Deploy to Heroku

1. Go to `wine_app` directory and create a file named `Procfile`. Don't give it any extension.
2. Add this to the file: `web: gunicorn wsgi:app`
3. Create `wsgi.py` with the following content: 

```python
##importing the app from main file
from main import app

if __name__ == '__main__': 
    app.run()
```

4. From `main.py` delete

```python
if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0', port=9696)
```

5. We installed some additional packages (Flask, gunicorn), so we need to update the requirements file. The easiest and the most complete way to do it is by running `pip freeze > requirements.txt`. You can also copy the list frpm below. Make sure that `env_wine_app` is activated before you do it.

```
certifi==2021.5.30
charset-normalizer==2.0.4
click==8.0.1
Flask==2.0.1
gunicorn==20.1.0
idna==3.2
itsdangerous==2.0.1
Jinja2==3.0.1
joblib==1.0.1
MarkupSafe==2.0.1
numpy==1.21.1
pandas==1.3.1
python-dateutil==2.8.2
pytz==2021.1
requests==2.26.0
scikit-learn==0.24.2
scipy==1.7.1
six==1.16.0
sklearn==0.0
threadpoolctl==2.2.0
urllib3==1.26.6
Werkzeug==2.0.1
```

7. Initialize `wine_app` as git repository and commit all files.

```bash
$ git init 
$ git add .
$ git commit -m "Initial Commit"
```

7. Go to Heroku and create a free account: https://signup.heroku.com/
8. Install Heroku locally: https://devcenter.heroku.com/articles/heroku-cli
9. Type in terminal: `heroku login`
10. Go to the browser and continue the login
11. Go back to the terminal: `heroku create give-me-best-wine` (the name of the application should be unique, so make up your own one)

```bash
(env_wine_app) ➜  wine_app git:(master) heroku create give-me-best-wine
Creating ⬢ give-me-best-wine... done
https://give-me-best-wine.herokuapp.com/ | https://git.heroku.com/give-me-best-wine.git
```

If we open the link, we will see that is nothing there yet.

12. Push the code to Heroku:

```bash
$ git push heroku master
```

It will take about a minute or two to install all the dependencies and deploy the app.

13. Test the endpoint from the notebook, the same way we did it for the local run, but change the URL to your link:
ex.: `url = "https://give-me-best-wine.herokuapp.com/predict"`
(`test_app_heroku.py`):

```python
new_data = {
    'fixed acidity': [1.2, 2.3, 1.3],
    'volatile acidity': [0.1, 0.8, 0.7],
    'citric acid': [0.5, 1.7, 1.1],
    'residual sugar': [3.0, 13.0, 7.6],
    'chlorides': [2.0, 15.0, 13.4],
    'free sulfur dioxide': [10.0, 72.7, 75.0],
    'total sulfur dioxide': [33.0, 135.0, 67.0],
    'density': [0.99, 1.007, 1.0],
    'pH': [3.0, 9.0, 6.7],
    'sulphates': [0.3, 1.8, 0.9],
    'alcohol': [9.3, 13.4, 12.2]
    }

import requests

url = "https://give-me-best-wine.herokuapp.com/predict"
r = requests.post(url, json = new_data)
r.text.strip()
print(r.text.strip())
```

## Data versioning with DVC

**Objectives:**

- Learn how to version a dataset
- Learn how to use a specific version of a data in your code
- Understand the interoperability between Git and DVC

**Dependencies:**

- dataset: `wine_original.csv`

**Instructions:**

1. [Install DVC](https://dvc.org/doc/install) inside `env_mlops`: `pip install dvc`
2. Copy the data file `wine_original.csv` to your computer (use `clone` or download the whole repository)
3. Execute the commands below. Make sure to understand what they do:
  - Create of DVC repo
  - Create of two different versions of the same dataset
  - Version of the metadata with Git
4. Go to your GitHub account and check out the tags. How would you add the descriptions to them to improve data cataloging?
5. Students: Create another version of the dataset the same way as it was shown. Pay attention to which files are being created and DVC/Git roles!
6. Install pandas, sklearn, jupyter notebook:
`pip install pandas`, `pip install sklearn`, `pip install notebook`
7. Extract the data version 2 (v2) in the code (wine model) with DVC API.

**Schema:**

![Lab work](./assets/lab_schema.png)

### Demo of data versioning with DVC

Create the directory:

```bash
mkdir data_versioning
cd data_versioning
```

Initialize the directory as git and dvc repository:

```bash
git init
dvc init
```

Check the new files that DVC created and commit them:

```bash
git status
git commit -m 'Initialize repository'
```

Copy the data file `wine_original.csv` to `data_versioning/data` and inspect its properties:

```bash
mkdir data
cp <path/to/the/wine_original.csv> data/
ls -l data/
```

Start tracking the dataset with DVC:

```bash
dvc add data/wine_original.csv
```

Check the files that appeared in `data/` (visible and hidden) and explore them with your favourite text editor or print the content to the terminal with `cat` command:

```bash
ls -l
ls -a
ls -l data/
ls -a data/
cat data/wine_original.csv.dvc
cat data/.gitignore
```

Track metadata with git:

```bash
git add data/.gitignore data/wine_original.csv.dvc
git status
git commit -m 'Add raw data'
```

Create DVC remote storage for the data. The actual data will be saved there.
For now, the dataset is still only in our folder. Metadata was created, but the dataset was
not pushed yet.

On the hard drive (local remote):

```bash
dvc remote add -d myremote /tmp/dvc-storage
```

or on Google drive (you will need to confirm by clicking on the link):

- go to your Google drive
- create a folder `dvc_data`
- open the folder and copy the part of the link after the last `/`:
  - example:
    - the full link: https://drive.google.com/drive/folders/1VYKTaf9fQvE5Jpif4B9gWVW33QDE7_aB
    - the part you need: 1VYKTaf9fQvE5Jpif4B9gWVW33QDE7_aB
- `pip install 'dvc[gdrive]'`

```bash
dvc remote add --default mydrive gdrive://<the part after the last `/` >
dvc remote add --default mydrive gdrive://1VYKTaf9fQvE5Jpif4B9gWVW33QDE7_aB
```

```bash
git commit .dvc/config -m "Configure remote"
git tag -a "v1" -m "raw data, v1"
git show
```

Now the remote storage is ready, so we can push the data.

```bash
dvc push
```

If you go to the remote storage, you will see the file appearing inside. The name is its hash code.
The correspondence between the original (human readable) name and the DVC-generated name is saved in the metadata. If we lose it, we will lose the linkage. Therefore, we need to version/track the metadata with Git.

```bash
# Chenge the link with the link to your own repository (make sure to keep .git at the end!)
git remote add origin https://github.com/pkaferle/mlops.git
git remote -v
git branch -M main
git push -u origin main
# Push tag (it doesn't happen automatically)
git push origin v1
```

Since we stored the data remotely, we can delete it locally.
DVC also stores the data in cache. We can delete it. Now we don't have any local copy of the dataset.

```bash
# Delete local version (don't delete .csv.dvc file!)
rm -rf data/wine_original.csv
# Delete from cache
rm -rf .dvc/cache
```

If we want to recover the data, we need to pull.

```bash
dvc pull
ls data
```

Let's modify the dataset to create the version 2 (v2). We will duplicate the whole content of the file.

```bash
cat <path/to/the/wine_original.csv> >> data/wine_original.csv
# Check the size, that the file is twice bigger (before it was cca. 84kB)
ls -l data
```

Now push the modified dataset to the remote storage and the metadata to GitHub.

```bash
dvc add data/wine_original.csv
git add data/wine_original.csv.dvc
git commit -m 'Add new data'   
git tag -a "v2" -m "append new data, v2"  
dvc push
git push
git push origin v2
```

Let's check the data about the different versions:

```bash
git tag
git show v2
git log
```

Now go to your GitHub repo and check out:

- The tags
- How you can add description and enrich the information about your dataset

**Exercise:** Create the v3 by deleting 200 lines and save it.

### Access the data through the [Python API](https://dvc.org/doc/api-reference)

Run the following code in a notebook or .py file.

```python
# Get url from DVC
import pandas as pd
import dvc.api

path='data/wine_original.csv'
repo='</path/to/data_versioning/folder>'

data_url = dvc.api.get_url(
  path=path,
  repo=repo
  )

data = pd.read_csv(data_url, sep=";")
print(data)
```

**Exercise:** Access a specific version of the dataset.

**External link:**

- video [Data Versioning and Reproducible ML with DVC and MLflow](https://www.youtube.com/watch?v=W2DvpCYw22o)

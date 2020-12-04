# A short overview of GitHub actions 

This is in no way meant to be a tutorial on GitHub actions on which there are already some nice options available:

- https://docs.github.com/en/free-pro-team@latest/actions/learn-github-actions
- https://github.blog/2020-06-17-using-github-actions-for-mlops-data-science/
- https://youtu.be/R8_veQiYBjI 

Instead this page gives a walkthrough of the `report.yml` which is the way in which the report GitHub action is defined.

## The YAML file

GitHub action workflows i.e. the steps an action does, are defined in a [`YAML`](https://en.wikipedia.org/wiki/YAML) file. Often workflows follow similar patters so it usually makes sense to copy an existing workflow which does similar things as a starting point.

### Triggering the action

GitHub Actions can be triggered by various different triggers (pushes to a repository, pull requests etc.) in this case we trigger and action on a schedule. This schedule can be defined using [Cron](https://en.wikipedia.org/wiki/Cron) syntax. [https://crontab.guru/]() is a great way of working out the syntax if you haven't used it before and helpfully the online GitHub editor also converts the syntax into a more readable format when you however over it.

In our case we run at 8:00 UTC every Monday. 

```yaml
name: Create Report
on:
  schedule: 
    # runs once a week on Monday at 8 UTC 
    - cron: "0 8 * * 1"
```

One the action has been triggered by an event, in this case a time, we need to define the next steps of the action.
### Setting up an environment

Once an action has been triggered you'll usually need to create some kind of environment to run the event. This can be fairly simple or very complicated depending on what you are trying to achieve. 

In this case the main things we need are a virtual environment, in this case running `ubuntu-20.04`, this is the  `runs-on`. We also use some externally defined actions `actions/checkout@v1` which checks out the repository we're running the action from, and `actions/setup-python@v1` which as the name suggests allows us to quickly use Python in our action.  

```yaml
jobs:
  build:
    runs-on: ubuntu-20.04
    steps:
    - uses: actions/checkout@v1
    - uses: actions/setup-python@v1
      with:
        python-version: '3.6'
        architecture: 'x64'
        ref: 'traffic'
```

This section installs the libraries we need to run the notebook. Since we have fairly simple requirements for this action we just pip install a few packages. If we had more complicated requirements we could specify the Python packages in a `requiremnts.txt` or similar.

```yaml
    - name: Install library and other requirements
      run: |
        pip install jupyter nbconvert altair 
        pip install git+https://github.com/Living-with-machines/gh_orgstats
   
```

The next section uses `nbconvert` to execute the report notebook and to convert it to html. 

One important thing to notice is `GH_TOKEN`, since the code in `report.ipynb` notebook requires an access token for GitHub we need to make that available. We could obviously include the token in our GitHub action but this would be a very bad idea since anyone with access to our GitHub repo could then see this token. Instead we store this token in a [GitHub secret](https://docs.github.com/en/free-pro-team@latest/actions/reference/encrypted-secrets). This is a way in which we can store tokens etc in an encrypted way. Access to the secrets can be limited to certain people meaning that our GitHub action can get to the token but random person from the internet can't. 

```yaml
    - name: Execute report notebook
      env:
        GH_TOKEN:  ${{ secrets.GH_TOKEN }}
      run: |
         jupyter nbconvert --to notebook --execute report.ipynb
         jupyter nbconvert --to html --TemplateExporter.exclude_input=True --no-prompt report.nbconvert.ipynb --output report.html
         mv *.html docs/
```

This final section commits the changed files back to GitHub. You can make this process much more complicated if needed i.e. commit to a different branch but in this case this isn't needed.

```yaml

    - name: Commit changes
      uses: EndBug/add-and-commit@v4
      with:
        author_name: Daniel van Strien
        message: "Update GitHub traffic report"
        add: "."
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }} 

 ```

There is potential for many more steps, to run as part of GitHub actions. The [official docs](https://docs.github.com/en/free-pro-team@latest/actions) are a good place to start for further information. 
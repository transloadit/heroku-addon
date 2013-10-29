Hi there,

thanks for taking an interest in our private beta!
Here are the steps to partake, and make sure it works using an example ruby app.

## Join the Heroku Beta program

http://beta.heroku.com/

## Get the `heroku` toolbelt 

At https://toolbelt.heroku.com/download/osx

## Login

Type `heroku login`

```bash
Email: kevin@vanzonneveld.net
Password: your-password
```

## Install a sample ruby app

```bash
git clone git://github.com/heroku/ruby-sample.git
cd ruby-sample
heroku create
git push heroku master
heroku open
```

## Add the Transloadit Addon

Type `heroku addons:add transloadit`

```bash
Adding transloadit on mysterious-inlet-1371... done, v4 (free)
Use `heroku addons:docs transloadit` to view documentation.
```

## More information

Please goto the [README.md](README.md), ask us anything on support@transloadit.com, 
send us pull requests, or reach out on [twitter](https://twitter.com/transloadit).


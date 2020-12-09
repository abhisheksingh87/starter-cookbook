+++
categories = ["recipes"]
tags = ["deploy cookbook","pcf"]
summary = "Deploy cookbook as a app in PCF"
title = "Deploy Cookbook to Pcf"
date = 2020-09-01T14:02:27-05:00

+++

## Prerequisite

- `cf CLI`
- `Hugo`

### Install instructions

#### cf CLI

* Install cf CLI from https://docs.cloudfoundry.org/cf-cli/install-go-cli.html

#### Hugo

* Install `Hugo` on your laptop from https://github.com/gohugoio/hugo/releases/download/v0.74.3/hugo_0.74.3_Windows-64bit.zip

* Update `PATH` to include the path to `hugo` executable


## Steps

1. Clone the **FedEx GSI** **cookbook** from Gitlab repo
   
    `git clone <cookbook repo>`
    
1. Open a command window and navigate to the `cookbook` directory

3. Start Hugo server _locally_ 

    `./localserver.bat` 

4. Check if **recipes** are displayed in the browser  (`http://localhost:1313`)

5. Add a new recipe or update an existing recipe(s)

6. **Publish** the cookbook with updated recipes: 

    `./publish`

7. cd  `<cookbook dir>/public`

8. **Deploy** the cookbook to PCF using CF cli

    `cf push`

1. **Verify** the cookbook with recipes is accessible at `https://<pcf API endpoint>/cookbook`


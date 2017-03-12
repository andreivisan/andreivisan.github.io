---
layout: post
title: Full step by step Amazon AWS Elasticsearch setup and usage
published: false
---

### Introduction

This tutorial shows you how to install and use Elasticsearch using Amazon AWS. For this we will use Amazon Elasticsearch Service (Amazon ES) to configure a domain. Elasticsearch is a very popular open-source and analytics engine and you can find more about it HERE. 
This tutorial can be completed using Amazon ES console, AWS CLI or the AWS SDK. In this tutorial I will use Amazon ES console, but I also recommend you have installed the AWS CLI.

### AWS CLI installation

1. Open <a href="https://console.aws.amazon.com/iam/home?#/home">IAM (Identity Access Management) console</a>.
2. In the navigation pane choose Users.
3. Choose your IAM user name.
4. Choose the Security Credentials tab and then choose Create Access Key.
5. To see your access key, choose Show User Security Credentials.
6. Choose Download Credentials and store the keys in a secure location (Your secret key will no longer be available through AWS. You will have the only copy).

Next we will install ```pip``` which is a package manager for Python that provides an easy way to install, upgrade and remove Python packages and their dependencies. If you already have Python installed check if you have version 3 of Python if you don’t then you need to install it. Please find bellow how to install Python 3 on Mac OS.

#### Python 3 installation

If you have Homebrew installed skip the next command, if not run it to install Homebrew on your machine:

```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

Now we can install Python 3.

```
$ brew install python3
```

#### Pip installation

1. Download the installation script from pypa.io:

```
curl -O https://bootstrap.pypa.io/get-pip.py
```

2. Run the script with Python:

```
python3 get-pip.py —user
```

3. Add the executable path to your PATH variable:

export PATH=~/.local/bin:$PATH

$ source ~/.bash_profile


4. Verify that ```pip``` is installed correctly:

pip3 --version

5. Install AWS CLI:

pip3 install awscli —upgrade —user

6. Verify that the AWS CLI installed correctly:

aws —version

#### Install AWS CLI with Homebrew on Mac OS

brew install awscli

aws —version

### Creating an Amazon ES domain

1. Go to https://aws.amazon.com. and then choose Sign In to the Console.
2. Under Analytics, choose Elasticsearch Service.
3. On the Define domain page, for Domain name, type a name for the domain.
4. Choose the version of Elasticsearch that you desire. Amazon recommends version 5.1.
5. Click on Next button.
6. For the instance count choose the number of instances you want.
7. For instance type choose an instance type from the Amazon ES domain. (I recommend t2.small.elasticsearch in case you just want to do some testing). You can learn more about Amazon WS free tier here: https://aws.amazon.com/free/
8. You can use Enable dedicated master or Enable zone awareness. For the purpose of this tutorial I didn’t enable them. You can read more about them here: http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/es-managedomains.html#es-managedomains-zoneawareness and http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/es-managedomains.html#es-managedomains-dedicatedmasternodes. 
9. If you want to use EBS volume storage, for Storage type, choose EBS.
    1. For EBS volume type choose the external storage type. For more information check here: http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSVolumeTypes.html
    2. For EBS volume size type the size in GB for the external storage for each data node. Use the following formula (number of data nodes) * (EBS volume size).
10. For Automated snapshot start hour, choose the hour of the day when automated snapshots will be taken.
11. (Optional) Choose Advanced options: 
    1. (Optional) If you want to configure access to domain sub-resources, for rest.action.multi.allow_explicit_index choose false.
    2. For indices.fielddata.cache.size specify the percentage of heap space to allocate to the field data cache. By default this setting in unbounded. For more information check here: https://www.elastic.co/guide/en/elasticsearch/reference/1.5/index-modules-fielddata.html
12. Choose Next.
13. Enter an access policy for the domain or select one of the policy templates from Select a template and then choose Next.
14. Review the new domain configuration and then choose Confirm and create.
15. Choose OK.
NOTE: It will take a few minutes until your ES instance is activated. Once is done move to the next step.

### Configuring an Access Policy for an Amazon ES Domain

1. Go to https://aws.amazon.com and then choose Sign In to the Console.
2. In the navigation pane, under My Domains choose the domain that you want to configure.
3. Choose Modify access policy.
4. Enter your current access policy or select one of the policy templates from Select a template and then choose Submit. For more information read here: http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/es-createupdatedomains.html#es-createdomain-configure-access-policies.

### Upload data to Amazon ES for indexing and searching

1. To add a single document use:

curl -XPUT search-movies-4f3nw7eiia2xiynjr55a2nao2y.us-west-1.es.amazonaws.com/movies/movie/tt0116996 -d '{"directors" : ["Tim Burton"],"genres" : ["Comedy","Sci-Fi"],"plot" : "The Earth is invaded by Martians with irresistible weapons and a cruel sense of humor.","title" : "Mars Attacks!","actors" : ["Jack Nicholson","Pierce Brosnan","Sarah Jessica Parker"],"year" : 1996}'

2. To upload a JSON file that contains multiple documents to Amazon ES domain:

curl -XPOST 'http://search-movies-4f3nw7eiia2xiynjr55a2nao2y.us-west-1.es.amazonaws.com/_bulk' --data-binary @bulk_movies.json

3. Using Node.js

If you use Node.js to connect to your Amazon ES cluster, simply use the URL given to you in your .env file as well as setting it in your environment variables in case you run your web app locally, then check the sanity of the cluster using the ES client. Find the code for that bellow:

```javascript
var elasticsearch = require('elasticsearch');
var client = new elasticsearch.Client({
    host: process.env.ELASTICSEARCH_TEST_URL,
    log: 'trace'
});

module.exports.pingElasticsearch = function(callback) {
    client.cluster.health({}, function(err, resp, status) {  
        if(err) {
            console.error('elasticsearch cluster is down!');
            callback(false);
        } else {
            console.log("-- Client Health --", resp);
            callback(true);
        }
    });
}

For more information about how to work with Elasticsearch and Node.js read <a href=“http://programminglife.io/searching-with-elasticsearch-and-node.js/” target=“_blank">here</a>.

I hope you found this article useful. If so please leave a comment bellow in the comments section.

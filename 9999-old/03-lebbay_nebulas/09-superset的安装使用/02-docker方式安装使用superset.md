> 参考： https://superset.incubator.apache.org/installation.html



```shell
git clone https://github.com/apache/incubator-superset/
cd incubator-superset
# you can run this command everytime you need to start superset now:
docker-compose up
```







1, 支持中文



vim 



```bash
pybabel update -i ./translation/messages.pot -d ./translations/ -l zh
```


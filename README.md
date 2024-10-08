# noctua-vulture
noctua man in the middle

<img width="808" alt="Noctua Vulture" src="https://github.com/user-attachments/assets/19698754-200d-45f3-9d81-b3f9d95579f7">

TIPS:
for enabling debugging need to set environment variable VULTURE_DEBUG=true

EASY START:

```javascript
import cluster from 'cluster';
import { availableParallelism } from 'node:os';
import process from 'node:process';
import { NoctuaVulture } from "noctua-vulture";
const numCPUs = availableParallelism();
const numThreads = numCPUs * 2;
if (cluster.isPrimary) {
    console.log(`Primary PID ${process.pid} is running, detected ${numCPUs} cpus, starting ${numThreads} threads...`);
    for (let i = 0; i < numThreads; i++) cluster.fork({THREAD_NUM:i});
    cluster.on('exit', (worker, code, signal) => { console.log(`thread PID ${worker.process.pid} died`,code,signal); });
} else {
    const server : NoctuaVulture = new NoctuaVulture();
    server.onRequest((ctx,callback)=>{
        let info = ctx.proxyToServerRequestOptions;
        if (info) console.log(info.method,info.host,info.port,info.path);
        callback();
    });
    server.onError((ctx,err,errorKind)=>{
        console.log(`Thread [${process.env.THREAD_NUM}] PID ${process.pid}`,ctx?.clientToProxyRequest.url,err,errorKind);
    })
    server.listen({host:"127.0.0.1",port:1982});
    console.log(`Thread [${process.env.THREAD_NUM}] PID ${process.pid} started`);
}
```
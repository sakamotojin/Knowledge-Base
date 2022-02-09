# Multithreading With LightWeight Threads

```
import json
import urllib.request
from urllib.parse import urlparse
from threading import Thread
import http.client, sys
from queue import Queue
from abc import abstractmethod, ABC

concurrent = 50
tasks = []

class Task(ABC):

    @abstractmethod
    def Init(self):
        raise NotImplementedError

    @abstractmethod
    def CleanUp(self):
        raise NotImplementedError

    @abstractmethod
    def CheckStatus(self):
        raise NotImplementedError

    @abstractmethod
    def Run(self):
        raise NotImplementedError
                

def doWork():
    while True:
        task = q.get()
        task.Init()
        task.Run()
        task.CleanUp()
        q.task_done()

#starting daemon threads 
q = Queue(concurrent * 2)
for i in range(concurrent):
    t = Thread(target=doWork)
    t.daemon = True
    t.start()
    
    
try:
    for task in tasks:
        q.put(task)
    q.join()
    
except KeyboardInterrupt:
    sys.exit(1)
```

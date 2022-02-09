# Logger

```python
import logging
from datetime import datetime


class Logger:
    __instance = None

    @staticmethod
    def getInstance():
        if Logger.__instance is None:
            Logger()
        return Logger.__instance

    def __init__(self):
        if Logger.__instance is not None:
            raise Exception("This class is a singleton!")
        else:
            Logger.__instance = self
            logging.getLogger("urllib3").setLevel(logging.WARNING)
            logging.root.setLevel(logging.NOTSET)
            logging.basicConfig(level=logging.NOTSET)
            self.errorLogger = logging.getLogger('VISUAL-SEARCH-ERROR')
            self.infoLogger = logging.getLogger('VISUAL-SEARCH-INFO')
            self.errorLogFile = 'logs/error/' + str(datetime.now().strftime('%Y-%m-%d %H:%M:%S'))
            self.infoLogFile = 'logs/info/' + str(datetime.now().strftime('%Y-%m-%d %H:%M:%S'))
            self.doInitialConfig()
            self.logInfo('LOGGER-STARTED')

    def doInitialConfig(self):
        infoHandler = logging.FileHandler(self.infoLogFile)
        infoHandler.setLevel(logging.INFO)
        infoFormat  = logging.Formatter('%(asctime)s - %(name)s - %(message)s')
        infoHandler.setFormatter(infoFormat)
        self.infoLogger.addHandler(infoHandler)

        errorHandler = logging.FileHandler(self.errorLogFile)
        errorHandler.setLevel(logging.ERROR)
        errorFormat = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
        errorHandler.setFormatter(errorFormat)
        self.errorLogger.addHandler(errorHandler)

    def logError(self, msg):
        self.errorLogger.error(msg)

    def logInfo(self, msg):
        self.infoLogger.info(msg)Py
```

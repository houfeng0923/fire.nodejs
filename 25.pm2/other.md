
```
pm2 start -n appname index.js
pm2 start -n appname npm -- run release

pm2 delete -s appname || :  // --silent and suppress error
```


- [pm25](https://github.com/PaulGuo/PM25/blob/master/README.md)
- [forever]()
- [trace](https://trace.risingstack.com/?utm_source=rsblog&utm_medium=sideblock&utm_campaign=trace&utm_content=Functional%20Reactive%20Programming%20with%20the%20Power%20of%20Node.js%20Streams)
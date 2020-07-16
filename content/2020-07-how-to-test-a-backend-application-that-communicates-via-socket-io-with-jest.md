# How to test a backend application that communicates via socket.io with jest

If you want to test a part of your node.js application that uses socket.io you basically have two options:

## Spin up a real socket.io server
This ensures that your application under test behaves like it would in production. But there are some drawbacks:
1. You test setup gets quite complex. You need to spin up a real socket.io server, connect it to your application and stop it when finished.
2. You need to create your own test framework. For example if you want to expect for an emitted value, you need to make it accessible for your tests.
3. Your test might be pretty hard to read for somewone who does not know what is going on.

```ts
function listenToNextEvent(eventName: string) {
  const socket = io(serverUrl);
  return new Promise((resolve) => {
    socket.on(eventName, (data: any) => {
      socket.off(eventName);
      resolve(data);
    });
  });
}

it('should send example data when some action is triggered', async () => {
  const exampleEventPromise: any = listenToNextEvent("example");
  httpClient.post(`${serverUrl}/trigger/some/action`)
  const exampleData: Example = await exampleEventPromise;
  expect(exampleData).toEqual("Hello World")
})
```

If you want to test a special behaviour like sending data on new client connections, the order is crucial.
```ts
it('should send example data when a client connects after some action is triggered', async () => {
  httpClient.post(`${serverUrl}/trigger/some/action`)
  const exampleEventPromise: any = listenToNextEvent("example");
  const exampleData: Example = await exampleEventPromise;
  expect(exampleData).toEqual("Hello World")
})
```

## Mock socket.io

Using the jest mock api results in a more readable test:
```ts
it('should send example data when some action is triggered', async () => {
  httpClient.post(`${serverUrl}/trigger/some/action`)
  expect(mockedSockets.sockets.emit).toHaveBeenCalledWith("example", "Hello World")
})
it('should send example data when a client connects after some action is triggered', async () => {
  httpClient.post(`${serverUrl}/trigger/some/action`)
  const newSocket = mockedSockets.triggerClientEvent("connection");
  expect(newSocket.emit).toHaveBeenCalledWith("example", "Hello World")
})
```
Therefore you need to create mocked sockets for each test and pass them somehow to your application. 
You might need to adapt the mocked socket implementation to your needs.
```ts
function createMockedSockets(): any {
  const registeredListener: {
    name: string;
    listener: (socket: Socket) => {};
  }[] = [];
  return {
    on: (name: any, listener: any) => {
      registeredListener.push({ name, listener });
    },
    sockets: { emit: jest.fn() },
    triggerClientEvent: (name: string) => {
      const socket: any = { emit: jest.fn() };
      registeredListener
        .filter((callback) => callback.name === name)
        .forEach((callback) => callback.listener(socket));
      return socket;
    },
  };
}
beforeEach(async () => {
  mockedSockets = createMockedSockets();
  new Application(mockedSockets);
});

```

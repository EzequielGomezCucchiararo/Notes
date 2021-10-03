# Angular

## Architecture planning

### Architecture considerations
There are a series of steps we should consider when we want to start a -not only Angular- application. 

It could be a good idea to present to the team via powerpoints -or similar tools- a slide for every of the next points and discuss with 
the team:

1. App overview: what is the app for? what is the goal? business and strategic benefits? Etc.
2. App features: list all the features we know at the moment
3. Domain security: what about authentication and authorization? are we using roles? groups? How is going to the angular communicate 
   with the api? via tokens?
4. Domain rules: Rules like validation will run client side? server side? both? Etc.
5. Logging: What do we do with the errors we caught?
6. Services/Communication: How is the front communicating with the server? HttpClient or Axios? NodeJs? REST or GrapQL? http only? web 
   sockets? How between front-end components itself?
7. Data models: What model data are we expecting in the front? How are going to be the interfaces? Are we going to use interfaces? And 
   classes? When to use inheritance and composition? How are going to be exactly?
8. Feature components: How are going to structure our components? Which one will be presentational or functional? Which components are 
   going to be lazy loaded? Which structure are they go to have? Are we going to use grids systems? What about UI libraries?
9. Shared functionality: Do we have shared functionality? any third-parties? Custom components? Could we reuse components between "apps"?
10. Test: unit test? behaviour test? e2e test? Which framework are we going to use? Which policy?
11. Coding rules: are we going to follow any style guide? Lint? Prettier?
12. Accessibility
13. i18n
14. Environments
15. CI/CD
16. CDNs, containers, servers...

Talking and planning about the above points will help us a lot to define a better -but not final- architecture.




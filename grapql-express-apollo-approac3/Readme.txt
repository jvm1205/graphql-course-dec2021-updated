In a new project, 
  npm install apollo-server-express apollo-server-core express graphql


Step 1:
---
in server.js 

  // Refer : How to use Apollo Server with Express JS 
  // https://www.apollographql.com/docs/apollo-server/integrations/middleware/
  // The following is for Apollo Server v3 - recently released.

  const express = require('express');
  const http = require('http');
  const { ApolloServer } = require('apollo-server-express');
  const { ApolloServerPluginDrainHttpServer } = require('apollo-server-core');

  const { typeDefs } = require('./Schemas/TypeDefs/TypeDefs');
  const { resolvers } = require('./Schemas/Resolvers/Resolvers');

  async function startApolloServer(typeDefs, resolvers) {
    // Required logic for integrating with Express
    const app = express();
    const httpServer = http.createServer(app);

    // Same ApolloServer initialization as before, plus the drain plugin.
    const server = new ApolloServer({
      typeDefs,
      resolvers,
      plugins: [ApolloServerPluginDrainHttpServer({ httpServer })],
    });

    // More required logic for integrating with Express
    await server.start();
    server.applyMiddleware({
      app,

      // By default, apollo-server hosts its GraphQL endpoint at the
      // server root. However, *other* Apollo Server packages host it at
      // /graphql. Optionally provide this to match apollo-server.
      path: '/',
    });

    // Modified server startup
    await new Promise(resolve => httpServer.listen({ port: 4000 }, resolve));
    console.log(`🚀 Server ready at http://localhost:4000${server.graphqlPath}`);
  }

  startApolloServer(typeDefs, resolvers);


Step 2: 
===
 Have Schemas/TypeDefs/TypeDefs.js
  
  const { gql } = require("apollo-server-express");

  const typeDefs = gql`

    #Custom Data Types
    type User{
      id: Int,
      name: String,
      phone: String
    }

    #Queries
    type Query {
      hello: String
      getUserList: [User]
    }

    type Mutation{
      createUser(name: String, phone: String): User 
      #name and phone are args in the above line
    }
  `;

  module.exports = {typeDefs};

Step 3:
=====
 Have Schema/Resolvers/Resolvers.js 

  const resolvers = {
    Query: {
      hello(){
        return 'Hey!'
      },
      getUserList(){
        // db query
        return [{
          id: 1,
          name: "Arun",
          phone: "234234"
        }]
      }
    },
    Mutation: {
      createUser(parent, args){
        console.log(args);
        return {
          id: 999,
          name: args.name,
          phone: args.phone
        }
      }
    }
  }

  module.exports = { resolvers };


Step 5:
===
  Try the following 
  
  query {
    getUserList  { 
      id
      name
      phone
    }
  }
  # if you are not happy with getUserList as prop -- you can use aliases. Refer the next query 


  query {
    users: getUserList  { #users is an alias - getUserList is a query 
      id
      name
      phone
    }
  }

Step :
====
  You can work on Mutations
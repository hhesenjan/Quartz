---
title: "Understanding JSON Web Token (JWT)"
created: 2024-10-25
modified: 2024-10-25
tags: ["NODEJS", "JWT"]
draft: false
---

## Introduction

Currently, I'm working on improving my programming skills therefore I decided to start programming an web application with authentication, authorization, and session handling. After researching for a while I stumbled upon JWT Token which I found fascinating and I decided to share the knowledge I gained to potentially help out others with their journey to learn JWT Token.

JSON Web Token (JWT) is commonly used for authentication and authorization as it's relatively small size and can be sent through a URL, POST Request, and inside a HTTP Header. The JWT Token contains inform-ation about the user in JSON format:

```json showLineNumbers
{
	"id" : "1",
	"name" : "Husenjan",
	"email" : "hhesenjan@hotmail.com",
	"role" : "User"
}
```

To avoid the data from being tampered with the JWT Token is encrypted and digitally signed. When an user tries to access a protected resource, the server extracts information inside JWT Token and determines what the user can access and cannot access.

## JWT Implementation

I'll be creating an Next.JS 14 web application and I'll be using the [Auth.JS v5](https://authjs.dev/getting-started) to implement JWT to our web application since it comes with all the features necessary for authentication and authorization. 

You will need to install the following modules:

```shell showLineNumbers
npm install next-auth@beta
npm install zod
```

You will need to generate a random value which will be used to encrypt the token:

```shell
npx auth secret
```

With Auth.JS v5 there are many providers which you can choose from such as Google, GitHub, Twitter, and Okta. However, I'll go with [Credentials Provider](https://authjs.dev/getting-started/authentication/credentials) as I'll authenticate against my own MySQL Server. We should start off setting up our input handler with the `zod` module.

```typescript showLineNumbers title="/libs/zod.tsx"
import { object, string } from "zod";

export const signInSchema = object({
  email : string({ required_error : "Email is required" })
  .min(1, "Email is too short"),
  password : string({ required_error : "Password is required"})
  .min(6, { message : "Password length must be minimum 6 characters"})
  .max(40, "Password cannot exceed 40 characters")
});
```

The `signInSchema` will be used to validate users input and prevent attacks such as SQL Injections, Server-Side Template Injections and so forth... Now as our users input are being handled, we can start implementing connection to MySQL server. 

```typescript showLineNumbers title="/libs/db.tsx"
import mysql from "mysql2/promise";

export const pool = mysql.createPool({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_DATABASE,
  waitForConnections: true,
});
```

You can replace the `process.env.*` with the database credentials and informations. Now we can start imple-menting JWT by creating `auth.tsx` inside our root directory.

```typescript showLineNumbers title="/auth.tsx"
import NextAuth from "next-auth";
import Credentials from "next-auth/providers/credentials"
import { signInSchema } from "./libs/zod";
import { pool } from "./libs/mysql";
const bcrypt = require("bcryptjs");

// Including `role` variable into JWT
declare module "@auth/core/types" {
  interface User {
      role?: string | null
  }

  interface Session {
      user : {
          role?: string | null | unknown
      }
  }

  interface Profile {
      role?: string | null
  }
}

declare module "@auth/core/jwt" {
  interface JWT {
      role?: string | null 
  }
}

// Handles the signIn, signOut, and sessions
export const {handlers, signIn, signOut, auth } = NextAuth({
  providers : [
    // Credentials allows us to authenticate against our own SQL server
    Credentials({
      credentials: {
        email: {},
        password: {},
      },
      authorize: async (credentials) => {
        // Validates the user input to ensure it follows our rules and doesn't contain malicious payload
        const vfields = await signInSchema.safeParseAsync(credentials);

        if (!vfields.success) {
          return { error : "Failed at validation" };
        }
        
        // Getting the user from database
        const db = await pool.getConnection();
        const query = "SELECT * FROM users WHERE email = ?";
        const [rows] = await db.query(query, [vfields.data.email]);
        db.release();

        // If user is not found it will fail the authentication
        if (!rows) {
          return null;
        }

        // If user is found then the password hash is compared to the password
        const comparePassword = await bcrypt.compare(vfields.data.password, rows[0].password);

        // If comparison fails it will fail the authentication
        if (!comparePassword) {
          return null;
        }
        
        // If authentication succeed it will return JWT Token with id, email, name, and role
        return {
          id : rows[0].id,
          email : rows[0].email,
          name : rows[0].name,
          role : rows[0].role,
        }
      }
    }),
  ],
  pages: {
      signIn: '/login',
  },
  callbacks: {
    // Implementing roles into our JWT Token
    async session({ session, token }) {
        session.user.role =  token.role;
        return session;
    },
    // Implementing roles into our JWT Token
    async jwt({ token }) {
        if (user) {
            token.role = user.role;
        }
        return token;
    },
  }
});
```

We can now start implementing our login page with the client mode to validare users input, handle errors, and authentication, and authorization. It's important to note that Auth.JS v5 supports authenticating thro-ugh client mode and server mode. In our case we will use client mode as we will handle multiple of error messages. 
```typescript showLineNumbers title="/components/sign-in.tsx"
"use client";
import { useRouter } from "next/navigation";
import { useState } from "react";
import { signIn } from "next-auth/react";
import { signInSchema } from "@/libs/zod";
import Link from "next/link";

// FormErrorHandler will be used to handle our validation errors
type FormErrorHandler = {
    email?: string[] | undefined, 
    password?: string[] | undefined,
}

export default function SignIn() {
  // Router will push the user to a specific page
  const router = useRouter();
  // useState will be used to handle the different errors
  const [validationErrors, setValidationErrors] = useState<FormErrorHandler>();
  const [commonErrors, setCommonErrors] = useState<string>();

  const credentialsAction = async(formData: FormData) => {
    // Validates user input
    const vfields = signInSchema.safeParse({
        email : formData.get("email"),
        password : formData.get("password"),
    });

    // Throws an error message if user input fails
    if (!vfields.success) {
        setCommonErrors(undefined);
        setValidationErrors(vfields.error.formErrors.fieldErrors);
        return;
    }
    else {
      // If validation is successfull it will send a sign-in request
      const res = await signIn("credentials", {
          email : formData.get("email"),
          password : formData.get("password"),
          redirect : false,
      });

      // If the sign-in fails an error message is displayed
      if (res?.error) {
          setValidationErrors(undefined);
          setCommonErrors("Invalid email or password.");
          return;
      }

      // If the sign-in succeed the user is pushed to /
      router.push("/");
    }
  }

  return (
      <div className="w-screen h-screen flex items-center justify-center">
          <div className="block w-[270px]">
              <div>
                  <p>Login</p>
              </div>
              <form action={credentialsAction}>
                  <div className="mt-2 grid grid-cols-1">
                      <label htmlFor="credentials-email" className="text-sm text-zinc-300">Email</label>
                      <input className="mt-1 p-1 pl-2 text-xs rounded-lg bg-zinc-900 outline outline-1 outline-zinc-700 text-zinc-300" type="email" name="email" />
                      
                      {  validationErrors?.email?.length !== undefined ? (
                          <p className="text-xs text-red-500 mt-2">{ validationErrors?.email }</p>
                      ) : null}

                  </div>
                  <div className="mt-2 grid grid-cols-1">
                      <label htmlFor="credentials-password" className="text-sm text-zinc-300">Password</label>
                      <input className="mt-1 p-1 pl-2 text-xs rounded-lg bg-zinc-900 outline outline-1 outline-zinc-700 text-zinc-300" type="password" name="password" />
                      
                      { validationErrors?.password?.length !== undefined  ? (
                          <p className="text-xs text-red-500 mt-2">{ validationErrors?.password }</p>
                      ) : null }
                      
                      { commonErrors?.length !== undefined ? (
                          <p className="text-xs text-red-500 mt-2">{ commonErrors }</p>
                      ) : null}
                  </div>
                  <div className="mt-2 grid grid-cols-1">
                      <button className="mt-1 text-sm p-1 bg-zinc-950 rounded-lg outline outline-1 outline-zinc-800 text-zinc-300">Sign In</button>
                  </div>
              </form>
              <div className="mt-1">
                  <Link className="text-xs text-sky-400" href="/register">Interested in signing up?</Link>
              </div>
          </div>
      </div>
  );
}
```

The `validationErrors` is used for displaying the error messages that comes from the `signInSchema`  and the `commonErrors` is used to displaying sign-in failed messages when users enters wrong credentials. If the login is successful then the user is pushed to the `/dashboard`. 

We can now start implementing our middleware to restrict access to specific protected resources such as `/dashboard` and `/admin` panels to logged in users.

```typescript showLineNumbers title="./middleware.tsx"
import { auth } from "@/auth";

export default auth( async(req) => {
  // Only allow administrators to access `/admin` panel
  if (req.auth && req.auth?.user.role === "Administrator" && req.nextUrl.pathname === "/admin") {
    return;
  }

  // If user is unauthenticated and tries to access any resources forward them to `/login`
  if (!req.auth && req.nextUrl.pathname === "/"){
    const newUrl = new URL ("/login", req.nextUrl.origin);
    return Response.redirect(newUrl)
  }
  
  // If user is authenitcated and tries to login forward then to `/dashboard`
  if (req.auth && req.nextUrl.pathname === "/login"){
    const newUrl = new URL ("/dashboard", req.nextUrl.origin);
    return Response.redirect(newUrl)
  }
})

// Avoids including middleware on the pages defined
export const config = {
  matcher: [
  '/((?!api|_next/static|_next/image|favicon.ico).*)',
  ],
}
```

Basically, all the middleware does is it ensures that only logged in users can access `/dashboard` and only administrators can access `/admin` panel. If you have specific resources that you only want specific roles to be able to access it's recommended to use middleware.

## Conclusion

Implementing JWT is fairly simple because of the [Auth.JS v5](https://authjs.dev/getting-started) library but there are some features missing such as a way to refresh tokens and delete token if it does get compromised by a malicious actor. However, these are security features which I'll go through implementing in the future once I'm more familiar with the Auth.JS v5 library.

Overall if you're not familiar with JWT, I highly recommend learning it by using the Auth.JS v5 library as it will help you with getting basic understanding about JWT.


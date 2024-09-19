## Discussion between Ken and Dinesh regarding email verification via DB lookup and Jwt tokens

Jet token vs db lookup:

kind of the strength here with JWT is that the token contains information that you don't have to fetch from a database. So  don't necessarily over index on avoiding the database calls.
It's like if you generated just a random token, and sent that without, you know, without a JWT token that you, you decode, you might actually save resources overall because in general it's cheaper to 
do a database look up than it is to do the cryptographic verification of jwt token.So you're just you're trading database traffic for CPU in the app servers. One nice thing is the CPU and the app 
servers is more horizontally scalable, so it could respond to an attack better. On the other hand Since we have such low normal usage rates, when we are in attack response mode, 
we could just, we could just suspend like this functionality completely, so.  You could also consider a database approach, and then lastly, when the system is overwhelmed, you'll have to 
figure out a way to fail in a way that because we don't want to fail in a way that gives a like a white screen, that's, that would, that should be avoided, and so figuring out a way to,
to fail in a way that lets someone know what's going on, that would be helpful 

So with the, with the jwts, you'll, you'll generate the JWT, you'll store information in the database like as part of that email validation, you know, kind of chunk. And then you'll, you'll, the, the token generation, it uses, a public key signature.you'll confirm the signature. And that, that public key signature verification check is expensive CPU wise.On the token, and then you'll email that thing off. When it comes back, you'll confirm the signature. And that, that public key signature verification check is expensive CPU wise. But if it fails or, you know, the, the data in the token is bad or The date stamp is expired et cetera. You won't need to fetch anything from the database, right? So that's it's really nice that way. On the other hand, verifying the token is really expensive, so you can just like fetch from the database, and, and, not have a token at all.
That's basically this this whole approach is, do you, do you send the information back to the client or do you just send an opaque token back to the client? So.
I don't have a strong preference either way, and I'm just saying that, it's, it's fine for you to approach jwts, jwts are probably more complicated and require more CPU resources, and so.

If you've already done that analysis and you're comfortable with that, that's fine. I'm just letting you know that if you wanted to have a simpler solution that that depends on the database more, that's also fine.


When implementing email verification, you have two main approaches: using JWT (JSON Web Tokens) or a database (DB) lookup. Here's a comparison of the two methods:

### 1. **JWT-Based Email Verification**
   - **How It Works**: You generate a JWT containing the user’s email (or ID) and send this token to the user’s email as part of the verification link. When the user clicks the link,
   - they send the token back to your server, which verifies the token’s signature and its validity.
   
   - **Pros**:
     - **Stateless**: No need to store verification information in the database, reducing DB load.
     - **Fast verification**: The server only needs to check the token's signature and claims, making the verification process faster.
     - **Expirable tokens**: JWTs can include an expiration time, ensuring that verification links expire after a certain period.

   - **Cons**:
     - **Security concerns**: JWTs are stateless, so if a token is compromised, it could be used without a chance for the server to revoke it unless other layers like blacklisting are implemented.
     - **Link manipulation**: If the token is too long-lived, someone could manipulate or misuse it if leaked.

   - **Best Use Case**: Ideal for low-complexity systems where minimizing database interactions is important, and the risk of token misuse is low.

### 2. **Database Lookup-Based Email Verification**
   - **How It Works**: After a user registers, you generate a unique token (or verification code) and store it in the database along with the user's information. The token is sent via email, and when the user clicks the verification link, your server looks up the token in the database, marks the user as verified, and removes the token.

   - **Pros**:
     - **Revocation control**: If necessary, tokens can be invalidated or removed at any time by deleting or altering the database record.
     - **Security**: Because each verification token is unique and stored server-side, the risk of token reuse or manipulation is lower.
     - **Trackable**: Since all verifications go through the database, you can track when users verify and manage expired or pending verifications.

   - **Cons**:
     - **Requires DB interaction**: Each verification requires a database lookup and update, which can introduce a small amount of overhead.
     - **Scalability**: If your system handles millions of verifications, the database load may increase, though modern databases can generally handle this with ease.

   - **Best Use Case**: Best for systems where security and token revocation are crucial, and where tracking user verification statuses is important.

### Summary
- **JWT-based verification** is good for **stateless**, fast, and scalable systems where you don't need to store verification records or revoke tokens.
- **DB lookup-based verification** is better for **security-focused** systems where revocation and precise tracking are important.

Which approach you choose depends on your system's requirements, such as scalability, security, and control over the verification process.

<img width="694" height="898" alt="image" src="https://github.com/user-attachments/assets/810f4cd9-747e-4cea-abb8-e732aab91831" />

# Initial recon
The challenge takes you to a website for a fictional band tour, when clicking the "**Buy Tickets**" button, we are redirected to a queue page:

<img width="929" height="552" alt="image" src="https://github.com/user-attachments/assets/9440042a-64f6-4fa6-ac47-f475b6af7831" />

We're prompted to join the queue by entering an email address. But once we do, the wait time is ridiculously long. Obviously, we want to skip the line.

<img width="769" height="518" alt="image" src="https://github.com/user-attachments/assets/f4ddff28-4d75-4a05-a594-b5db132af31f" />

# Queue logic

While inspecting the requests in burp, I noticed that submitting an email triggers the generation of a JWT token, which is then sent to the `/check_queue` endpoint. This endpoint returns your queue time based on the token.

When decoded, the JWT structure looks like this:
```json
{
  "user_id": "email@site.com",
  "queue_time": 1234,
  "exp": 111111111
}
```
The JWT uses HS256 as the signing algorithm. I'd have to figure out how to forge a token somehow.

# Discovering the signing secret
I tried sending garbage data to the `/check_queue` endpoint in the token parameter and got lucky. The server threw a 500 error that leaked a ton of useful info, including the JWT signing secret:

<img width="1267" height="603" alt="infinite_queue" src="https://github.com/user-attachments/assets/d2cf4397-c10f-4943-86ea-b5def4d94254" />

# Forging the token
Using the leaked secret, I forged a valid JWT with a much lower queue time.

<img width="569" height="240" alt="jwt" src="https://github.com/user-attachments/assets/6e4d3635-4906-4f94-b723-5a0536757097" />

This successfully skips the queue and gives you the flag!

# Challenge link
URL:      [http://184.72.87.9:8012](http://184.72.87.9:8012) (maybe the server was shut down)

# Challenge description
This is a very basic website where you can view other user's profiles, but you can only see your own secret. I'll bet other users' secrets have something of interest

# Challenge source file
Blackbox (No source)

# Difficulty
Very easy

# Approach
- When I connected to the website, all I saw was just a simple page:
  ![image](https://github.com/NoSpaceAvailable/SquareCTF/assets/143888307/5b254bde-b4e5-4596-83a3-6ceb73a4ca5b)

- I tried to click on the *CTF Participant* and *Admin* buttons and saw:
  ![image](https://github.com/NoSpaceAvailable/SquareCTF/assets/143888307/357a4ad8-596b-4885-8db8-56d1b3d31cef)
  ![image](https://github.com/NoSpaceAvailable/SquareCTF/assets/143888307/f940f5db-ab26-4a3d-823f-202fa90cbc05)

- So I need to become an admin. Let's check the cookies:
  ![image](https://github.com/NoSpaceAvailable/SquareCTF/assets/143888307/90704249-e501-4abe-b33d-38df674c22a7)

- According to my experience, let's try to decode it with basic algorithm. The first is base64:
  ![image](https://github.com/NoSpaceAvailable/SquareCTF/assets/143888307/a06afe28-347f-47b8-a3a4-7b52ad0262cf)

# Problem-solving
  ## Change cookie to gain admin access
  - I encoded *Admin* as base64 and put it in the cookie field, then save and reload page:
    ![image](https://github.com/NoSpaceAvailable/SquareCTF/assets/143888307/0b0d0b5e-ed8b-4fbf-be9d-59e2ec1b896e)

  - Flag: flag{boyireallyhopenobodyfindsthis!!}

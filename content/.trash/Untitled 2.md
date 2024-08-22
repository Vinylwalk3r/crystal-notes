Okay, I think I have a rough idea here. First of *disclaimer time* I'm not UI designer at all. So I just throwing a rough and unpolished idea here. But lets hope it 

can inspire something great!

- So, for my first idea, in the "x" menu, when having a container selected, this could be an option:
![X Menu](https://github.com/user-attachments/assets/637749a0-1857-4fc7-b22f-ece9909278d3)
If chosen, a big text box could appear which allows all settings (from either it's compose code or run command) could be edited. See below for a rough example of such a text box

- If you select a image and go to "x" -> "Run Image", instead of it just asking for a container name and then running, it could ask for a name, then ask if you want to input a compose code or run command:
![Input your Docker Compose or Run](https://github.com/user-attachments/assets/c2d12f52-ea76-441c-a11c-10cf253119f2)

I've been thinking, do you think Isaiah could handle compose codes? It would need to 
1. "mkdir new-compose-project" then 
2. "touch (or whatever command is necessary" docker-compose.yml "all the inputted compose code" 
3. then "docker compose up"
Would that be possible, in whatever implementation you choose?

- After the compose / run selection screen, a text box could appear that allows you to input your code / command:
![Input Compose Codes](https://github.com/user-attachments/assets/fbfcf6d1-70d6-4a52-8e51-8e6f8c84cf68)
After this, it runs as usual.

Do you think any of this could be doable / fitting for Isaiah?

systemctl status isaiah.service
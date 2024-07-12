So the telegram member scraping and adding script is pretty much easy.

It’s a nice opportunity to get attentions from related Telegram groups. You may want to scrape members from other related groups and add them to yours. Also, you can send messages to them and start engaging (Without Spamming!)

Let's get started 

Step 1
Create a Telegram App and Get Your Credentials
Go to my.telegram.org  and log in.
Click on API development tools and fill the required fields.
You can choose any name for your app. After submitting, you will receive api_id and api_hash. Save them somewhere. You will use these credentials to login to Telegram API.

Step 2
Install Telethon
Telethon is a  great MTProto API Telegram client library written by LunamiWebs, you can check the Github page here. You can install telethon using pip:

To install telethonPython
 pip install telethon
NB: Make sure python is already installed in your computer before attempting to install telethon, otherwise you will be graced with a massive error. Also if you are on Linux or Mac, you might need to use sudo before pip to avoid permissions issues.

Step 3

Create Client Object and Login
Latest version of telethon has two sync and async modules. The async module is using asyncio which is out of the scope of this article. Although you can get the same functionality using both modules but for the sake of simplicity we will use the sync module  in this tutorial.

So as the first step you need to import the sync module from Telethon library.

from telethon.sync import TelegramClient
 
Then, instantiate your client object using the credentials you got before.

from telethon.sync import TelegramClient
api_id = 123456
api_hash = 'YOUR_API_HASH'
phone = '+111111111111'
client = TelegramClient(phone, api_id, api_hash)
 
Next step would be connecting to telegram and checking if you are already authorized. Otherwise send an OTP code request and ask user to enter the code they received on their telegram account.

client.connect()
if not client.is_user_authorized():
    client.send_code_request(phone)
    client.sign_in(phone, input('Enter the code: '))
 
After logging in, a .session file will be created. This is a database file which makes your session persistent.

Step 4


Listing All Telegram Groups
Create an empty list for chats and populate with the results which you get from GetDialogsRequest. You need to import two more functions:  GetDialogsRequest and  InputPeerEmpty

from telethon.tl.functions.messages import GetDialogsRequest
from telethon.tl.types import InputPeerEmpty
chats = []
last_date = None
chunk_size = 200
groups=[]

result = client(GetDialogsRequest(
             offset_date=last_date,
             offset_id=0,
             offset_peer=InputPeerEmpty(),
             limit=chunk_size,
             hash = 0
         ))
chats.extend(result.chats)
 
Note: offset_date and  offset_peer are used for filtering the chats. We are sending empty values to these parameters so API returns all chats. offset_id and limit are used for pagination. Here we are getting last 200 chats of the user.

In this tutorial, we assume that we are only interested in mega groups; so check if the megagroup attribute of the chat is True and add it to your list.

This is warped inside a try, except block avoid getting an error, because some chats do not have megagroup attribute at all. So basically we skip those using continue statement.

for chat in chats:
    try:
        if chat.megagroup== True:
            groups.append(chat)
    except:
        continue

Step 5

Ask User to Select a Group to Scrape Members


After listing the groups, prompt the user to input a number and select the group they want. When this code is executed it loops through every group that you stored in previous step and print it’s name starting with a number. This number is the index of that is your group list.
 
i=0
for g in groups:
    print(str(i) + '- ' + g.title)
    i+=1
Ask user to enter a number associated with a group.  Then use this number as index to get the target group.
g_index = input("Enter a Number: ")
target_group=groups[int(g_index)]
 

Step 6

Get All Telegram Group Members
The last but not least step is to export all members (participants) of the Telegram group. Fortunately, there is a function for this in Telethon library which makes our job really simple.

Create an empty list of users and get members using the  get_participants  function and populate the list.
print('Fetching Members...')
all_participants = []
all_participants = client.get_participants(target_group, aggressive=True)
Important: Set the aggressive parameter to True otherwise you will not get more than 10k members. When aggressive is set to true, Telethon will perform an a-z search in the group’s participants and it usually extracts more than 90% of the members.

Step 7
Now use Python’s csv module to store the scraped data in a CSV file. First open a csv file in the write mode with an UTF-8 encoding. This is important because users might have non ASCII names which is very common in Telegram groups. Then create a CSV writer object and write the first row (header) in the CSV file. Finally, loop through every item in the all_participants list and write them to the CSV file.

print('Saving In file...')
with open("members.csv","w",encoding='UTF-8') as f:
    writer = csv.writer(f,delimiter=",",lineterminator="\n")
    writer.writerow(['username','user id', 'access hash','name','group', 'group id'])
    for user in all_participants:
        if user.username:
            username= user.username
        else:
            username= ""
        if user.first_name:
            first_name= user.first_name
        else:
            first_name= ""
        if user.last_name:
            last_name= user.last_name
        else:
            last_name= ""
        name= (first_name + ' ' + last_name).strip()
        writer.writerow([username,user.id,user.access_hash,name,target_group.title, target_group.id])      
print('Members scraped successfully.')

Note 1: Not every user has a username. If the user has no user name the API will return None. To avoid writing None and instead of writing an empty row, check if the user has a user name; otherwise, create an empty string as the username.

Note 2: Similar to the username, some user might don’t have a first name or last name, so we are doing the same thing for name as well.

For some large groups it might take a few minutes to get the members. But finally you should see this message  Members scraped successfully.  which shows that everything worked perfectly.

Note 3: I recommend using VScode to run the python script if you're unfamiliar with the cmd interface and also make sure you're connected to the internet.

Now the second phase of the code will also ask you for which group to add the members into, I combined everything inside.

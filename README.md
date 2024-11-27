# Django Video Chat Application

This repository contains a Django application for managing a video chat platform. The application provides features like user authentication, notification management, real-time chat, and room management using Agora's RTC token builder.

---

## Features

1. **User Authentication**:
   - Login and registration with error handling for existing users and invalid credentials.
   - Password hashing and confirmation during registration.
   - User session management.

2. **Notifications**:
   - API endpoint for creating notifications.
   - Displaying notifications with counts in the UI.

3. **Translation**:
   - Support for setting and activating user language preferences.

4. **Video Chat Integration**:
   - Agora RTC token generation for real-time communication.
   - Room member creation and deletion.

5. **Discussion Management**:
   - API for adding posts to discussions.
   - Notifications for new posts.

6. **Room Management**:
   - Create and retrieve room members.
   - Delete members from rooms.

---

##Installation
###Clone the repository:
git clone https://github.com/your-username/your-repo.git
###Install dependencies:
pip install -r requirements.txt
###Configure your .env file:
APP_ID=your-agora-app-id
APP_CERTIFICATE=your-agora-app-certificate
###Run migrations:
python manage.py migrate
###Start the server:
python manage.py runserver 
## Code Overview

### Token Generation
```python
from agora_token_builder import RtcTokenBuilder
import random
import time

def getToken(request):
    appId = 'your-app-id'
    appCertificate = 'your-app-certificate'
    channelName = request.GET.get('channel')
    uid = random.randint(1, 230)
    expirationTimeInSeconds = 3600 * 24
    currentTimeStamp = time.time()
    privilegeExpiredTs = currentTimeStamp + expirationTimeInSeconds
    role = 1
    token = RtcTokenBuilder.buildTokenWithUid(
        appId, appCertificate, channelName, uid, role, privilegeExpiredTs
    )
    return JsonResponse({'token': token, 'uid': uid}, safe=False)

from django.contrib.auth import authenticate, login as auth_login, logout

@csrf_exempt
def login(request):
    if request.method == 'POST':
        username = request.POST.get('email')
        password = request.POST.get('password')
        user = authenticate(request, username=username, password=password)
        if user:
            auth_login(request, user)
            request.session['username'] = username
            messages.success(request, 'Login successful')
            return redirect('home')
        else:
            messages.error(request, 'Invalid login details')
            return redirect('welcome')
    return render(request, 'base/welcome.html')

def logout_view(request):
    logout(request)
    return redirect('login')
@csrf_exempt
def create_notification(request):
    if request.method == 'POST':
        data = json.loads(request.body)
        sender = data.get('sender')
        body = data.get('body')
        Notifications.objects.create(sender=sender, body=body)
        return JsonResponse({'status': 'Notification created successfully'}, status=201)
    return JsonResponse({'error': 'Invalid request'}, status=400)
@csrf_exempt
def createUser(request):
    try:
        data = json.loads(request.body)
        member, created = RoomMember.objects.get_or_create(
            name=data['name'], uid=data['UID'], room_name=data['room_name']
        )
        return JsonResponse({'name': data['name']}, safe=False)
    except Exception as e:
        return JsonResponse({'error': str(e)}, status=500)

@csrf_exempt
def deleteMember(request):
    data = json.loads(request.body)
    member = RoomMember.objects.get(
        name=data['name'], uid=data['uid'], room_name=data['room_name']
    )
    member.delete()
    return JsonResponse('Member was deleted', safe=False)
@api_view(['POST'])
def add_post(request):
    data = request.data
    post = Post.objects.create(
        sender=request.session['username'], body=data['body']
    )
    Notifications.objects.create(
        sender=request.session['username'],
        body=f'New message from {request.session["username"]} in group chat',
    )
    serializer = PostSerializer(post, many=False)
    return Response(serializer.data)
---



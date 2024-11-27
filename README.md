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

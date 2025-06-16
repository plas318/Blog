## Views

The view will decide what page you would show, which request methods u will allow, what response you will respond and etc..

There are two primary ways of setting up the view, CBV , FBV 

* Similar to React
*     Class based hooks -> Function based hooks as good practice
* Django recommends
*     FBV -> CBV as good practice



​### HTTP Methods

- Standard : GET, POST, PUT, PATCH, DELETE
​

REST :: Represential State Transfer

​


### Function (Template) View

Simple Function View example:

```
def register(request):

    if request.method == 'POST': # request comes from the http standard, the POST method requires a form with a body
        form = UserRegisterForm(request.POST) # 

        if form.is_valid():
            username = form.cleaned_data.get('username')
            messages.success(request, f'Successfully created Account {username}!')
            form.save() # Commits the new User object based on form
            return redirect('blog-home')            
        
    else:
        form = UserRegisterForm()
    # Default method는 GET
    return render(request, 'users/register.html', {'form' : form} )
```

Use if statements and request.method == 'GET', 'POST', 'PATCH' to manipulate different behaviours



​

### ClassBasedView APIView

```
class LoginView(APIView):
    '''
    Login View

    - post: Login user based on the credentials
    '''
    
    authentication_classes = [SessionAuthentication] # Session Based Authentication
    permission_classes = [AllowAny]

    def post(self, request):
        serializer = LoginSerializer(data=request.data, context={'request':request})
        
        if serializer.is_valid(raise_exception=True):

            user = serializer.validated_data['user']
            login(request, user)
            return Response(status=status.HTTP_202_ACCEPTED)
        else:
            return Response(serializer.errors, status=status.HTTP_401_UNAUTHORIZED)
```

Class Based View uses the function to define the crud behaviours
request.method == 'post' -> def post(self, request): 

The LoginSerializer takes the login function from django.contrib.auth and verifies the user -> saving the session data and csrf token

Instead of returning an entire template page, a restful architecture returns a Json Response containing just the data like -ajax

Once the data is sent through json, the frontend renders the view page based on the data.


## ViewSets

Viewset basically groups up the functions for the view

the login view above only allows one method from the user : POST request to login,

however many pages are going to require multiple methods to be allowed for the user e.g) post page is going to allow the user to 

GET: list all the posts
POST: write a new post
UPDATE: edit a post
DELETE: delete a post

and such, viewset would group all of the behaviours for a specific page and manage it DRY-wise

```
class UserViewSet(viewsets.ModelViewSet):
    '''
    User viewset
    '''
    queryset = User.objects.all()
    serializer_class = UserSerializer
    filter_backends = [SearchFilter, OrderingFilter]
    search_fields = ['name', 'email']
    ordering_fields = ['name', 'email']
    ordering = ['name']

    http_method_names = ['get', 'update', 'patch', 'delete']
    
    def get_permissions(self):
        actions = ['update', 'partial_update', 'destroy']

        if self.action in actions:
            permission_classes = [IsAuthenticated & (isOwner|isAdmin)]
        else:
            permission_classes = [IsAuthenticated]

        return [permission() for permission in permission_classes]

    def destroy(self, request, *args, **kwargs):
        '''
        Override Destroy Method to Inactivate user on delete request
        '''
        instance = self.get_object()
        instance.is_active = False
        instance.save()
        return Response({'detail':'Deleted User'}, status=status.HTTP_204_NO_CONTENT)
```

To define a viewset, I've inherited the modelviewset provided by django.

queryset defines all the data to be worked with from the viewset, since this is the user class, I'm fetching all the users.

serializer_class defines the serializer to be used within the viewset, different functions within the viewset might want to override this class to user a specific serializer.

filter_backend is a field provided by the djangobackendfilter class -> allows the user to filter based on the criterias'

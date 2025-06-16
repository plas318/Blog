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

filter_backend is a field provided by the drf packages' .filters class -> allows the user to filter based on the criterias
search_fields, order_fields provides options to search, and order the query results

e.g) posts/?search=moon&ordering=modified_date

http_method_names : allows to define the http methods this View accepts

get_permissions is a function defined within the drf package,

I've orverrided the function in order to permit certain actions (methods) based on the user's state of authentication and authority.
i.e) checks if user is an admin, or the owner of the post to give update, delete access.

I've also overrided the destroy function -> (delete) method function to not remove the user instance, but to change it's state to "intactive"
this way the db doesn't suffer from having to cascade a lot of data and causing performance to suffer in some cases


### Generic View

So the viewset above gives a nice group of methods to provide all the CRUD methods to the user,
however alot of the cases, you would want to provide only a couple of methods.

e.g) A ListCreateAPIView only provides users the method to retrieve and create posts
you don't want to give them access to the update, delete methods since they might not be the author of the post.

e.g) RetrieveUpdateDestroyView on the other hand, is well suited for the user whom is the owner of the post, to give them the authority to update or delete their own posts. 

Also a good way to seperate the query instance too.
The list method should work with a series of posts, where retrieve, update, destroy methods are working with a single post instance.


### API View

API view is the most fundamental flexible customized function where you can define all the behaviours for a special view.
It is more tedious to create since you have to write all the code and perhaps repeat code for situations where a generic view or viewset might exist.

```
class SalesHistoryListView(APIView):
    '''
    SalesHistoryView List, Post View

    *query performance differs by using select_related, and prefetch related
    '''

    @extend_schema(
    methods=['get'],
    parameters= [
        OpenApiParameter('hospital_id', OpenApiTypes.STR, OpenApiParameter.QUERY, required= False, description='Filters result based on hospital id(요양기호) e.g. JDQ4MTAxMiM1MSMkMSMkMCMkODkkMzgxMzUxIzExIyQyIyQzIyQwMCQyNjE0ODEjNjEjJDEjJDgjJDgz '),
        OpenApiParameter('page', OpenApiTypes.INT, OpenApiParameter.QUERY, required= False, description= 'Returns result based on page, default=1, items_per page=20'),
        OpenApiParameter('ordering', OpenApiTypes.STR, OpenApiParameter.QUERY, required= False, description='Orders result based on fields | available fields: modified_at, (요양기호)hospital, status, (saleshistory)id'),
    ],
    responses={200: SalesHistorySerializer(many=True)},
    # more customizations
    )
    def get(self, request):
        '''
        no parameter:
        
        list: returns The entire list of sales history, ordered by modified date, and hospital id
        
        parameter hospital:
        
        list: returns The filtered list of saleshistory on a hospital id
        params: hospital (hospital_id: str)

        parameter page
        '''
        
        hospital = request.query_params.get('hospital', None)
        
        if hospital is not None:
            queryset = SalesHistory.objects.filter(hospital=hospital)\
                                          .order_by('-modified_at', 'hospital')\
                                          .select_related('hospital__manager', 'hospital__director')
        else:
            queryset = SalesHistory.objects.all().order_by('-modified_at', 'hospital')\
                                                .select_related('hospital__manager', 'hospital__director')
        
        page = request.query_params.get('page', 1)
        ordering = request.query_params.get('ordering', None)

        if ordering is not None:
            ORDERING_FIELDS = [
                'modified_at', '-modified_at',
                'hospital', '-hospital',
                'id', '-id',
                'status', '-status',
                'hospital_name', '-hospital_name'
                ]
            if ordering in ORDERING_FIELDS:
                if ordering == 'hospital_name':
                    ordering = 'hospital__' + ordering
                elif ordering == '-hospital_name':
                    ordering = '-hospital__' + ordering
                    
                queryset = queryset.order_by(ordering)
        
        if page is not None:
            from django.core.paginator import Paginator
            paginator = Paginator(queryset, 20)  # 20 per page
            queryset = paginator.get_page(page)

        serializer = SalesHistorySerializer(queryset, many=True)
        return Response(serializer.data, status=status.HTTP_200_OK)

```







**Power of the IOTA Cryptocurrency**

<p>Have you ever seen Office Space the movie? I was inspired initially as strange as it is, by that movie. 
The protagonists in the move write software to reroute money that is a rounding error caused by banking transfer and generic payments to the company. This seems like it would not be a lot of money, but in just a few weeks they were able to amass hundreds of thousands of dollars. Thats a lot! This is a prime reason why a free form platform for micropayments is essential for exchanging payments for services that have infinitesimal cost. The physical cost of a transaction on centralized platform will exceed the payment for the service thus forcing either over purchase or no purchase at all for a given microservice. This is how IOTA will disrupt the current payment system and allow free instantaneous exchange of micropayments without having to worry about fees. Most importantly, you wont be charged extra money because you didn't meet some sort of minimum denomination and no forcible purchases beyond the need of the customer. We plan to highlight much more of the potential of this technology in future posts. It goes way beyond this simple POC Chris and I have set up.<\p>


<p>Past my inspiration, I was able to collaborate with Chris Dukakis and come up with a POC that exchanges weather data for infinitesimal payments INCLUDING ZERO. Below there will be code snippets to highlight these features and I will walk you through how they all work. Let's proceed.<\p>

<p>Here is the first code snippet, with comments included to help you along the way. Chris has graciously set up a server to generate new API keys if you dont have one and he has also set up a tool to return the address, price, and transaction tag of a potential request. After we generate the new API (if you dont already have one), we initiate a request for the payment and add the transactions to the tangle. All the code inside the payment request if-else-block will be explained as you scroll down.<\p>

{% highlight python %}
def requestData(api_key=None):
    if api_key is None:
        key = generate_api_key()
    # type: () -> Optional[Text]
    request_id = TryteString.from_string(str(register_request()))[0:27]

    # :todo: Populate these values and/or make API object globally-accessible.
    request_dict = create_request(headers={"Authorization": key})
    response_headers = request_dict['request'].headers
    price = response_headers['price']
    accept_payment = input("The server asks for a payment of " + price + " IOTAs. proceed? Y/N\n")
    if accept_payment == 'Y':
        print("Thank you for your payment the data will now be served...")
        transaction = create_transaction_dictionary(address=response_headers['address'],
                                                    price=response_headers['price'],
                                                    request_tag=response_headers['tag'])
        api = create_iota_object(uri='http://85.93.93.110:14265/', auth_token=None)
        st_response = api.send_transfer(
            depth=transaction['depth'],

            # One or more :py:class:`ProposedTransaction` objects to add to the
            # bundle.
            transfers=[transaction['transaction-object']],
        )

        # Extract the tail transaction hash from the newly-created
        # bundle.
        bundle = st_response['bundle']  # type: Bundle
        request_dict = create_request(headers={'transaction': str(bundle.tail_transaction.hash)[18:-2],
                                               "Authorization": key})
        return request_dict['request'].json()
    else:
        print("You have not agreed to pay for the request data, so a an empty object will be returned")

    return None
{% endhighlight %}

<p>Now we have the method to get a new API key from Chris' server. It is very simple. All we do is make i get request for a new API key.<\p>
{% highlight python %}
def generate_api_key():
    r = requests.get("http://46.101.109.238/new_api_key")
    return r.content.decode("utf-8")
{% endhighlight %}

<p>From the headers returned from the server we proceed to create the transaction dictionary and an IOTA object instantiation to send the transfer if the end user has agreed.<\p>
{% highlight python %}
def create_transaction_dictionary(address, price, depth=3, request_tag=None):
    # type: (int, Optional[binary_type]) -> dict

    # encode the string so it doesnt cause conflict during encode/decode process

    r_tag=Tag(request_tag.encode('utf-8'))
    sample_transaction = ProposedTransaction(
        # on the fly API payment address.
        address=Address(str.encode(address)),
        # Amount of IOTA to transfer.
        # This value may be zero.
        value=int(price),

        # Optional tag to attach to the transfer.
        tag=r_tag,

        # Optional message to include with the transfer.
        message=TryteString.from_string('I am making an API Request!'),
    )
    return {"depth": depth, "transaction-object": sample_transaction, "tag": r_tag}
    
def create_iota_object(uri, auth_token, seed=None):
    # type: (Text, Text, Optional[Seed]) -> Iota
    return Iota(
        # To use sandbox mode, inject a ``SandboxAdapter``.
        adapter=SandboxAdapter(
            # URI of the sandbox node.
            uri=uri,

            # Access token used to authenticate requests.
            # Contact the node maintainer to get an access token.
            auth_token=auth_token,
        ),

        # Seed used for cryptographic functions.
        # If null, a random seed will be generated.
        seed=seed,
    )
{% endhighlight %}

<p>If you noticed the seed argument for IOTA obj instantiation you can create a new seed/address combination using the following:<\p>

{% highlight python %}
def generate_addresses(index=0, uri='ip address', auth_token=None):
    # type: (int, Text, Text) -> Address
    seed = get_seed()

    # Create the API instance.
    # Note: If ``seed`` is null, a random seed will be generated.
    if auth_token is None:
        api = create_iota_object(uri=uri, auth_token=auth_token, seed=seed)
    else:
        api = create_iota_object(uri=uri, auth_token=auth_token, seed=seed)

    # If we generated a random seed, then we need to display it to the
    # user, or else they won't be able to use their new addresses!
    if not seed:
        print('A random seed has been generated. Press enter to generate it.')
        output_seed(api.seed)

    print('Generating addresses.  Please wait until the address is generated...')
    print('')

    # generate address based on count arg
    gna_result = api.get_new_addresses(index, 1)
    print('address is now generated')
    return gna_result['addresses'][0]


def get_seed():
    # type: () -> binary_type
    """
    Prompts the user securely for their seed.
    """
    print(
        'Enter seed and press return (typing will not be shown).  '
        'If empty, a random seed will be generated and displayed on the screen.'
    )
    seed = secure_input('')  # type: Text
    return seed.encode('ascii')


def output_seed(seed):
    # type: (Seed) -> None
    """
    Outputs the user's seed to stdout, show security warnings from boiler plate code (Obligatory PSA)
    """
    print(
        'WARNING: Anyone who has your seed can spend your IOTAs! '
        'Clear the screen after recording your seed!'
    )
    compat.input('')
    print('Your seed is:')
    print('')
    print(binary_type(seed).decode('ascii'))
    print('')

    print(
        'make sure to clear the screen contents',
        'and press enter to continue.'
    )
    compat.input('')
{% endhighlight %}

<p>After the following codeblock for the transaction in the requestData method:<\p>
{% highlight python %}
st_response = api.send_transfer(
            depth=transaction['depth'],

            # One or more :py:class:`ProposedTransaction` objects to add to the
            # bundle.
            transfers=[transaction['transaction-object']],
        )

        # Extract the tail transaction hash from the newly-created
        # bundle.
bundle = st_response['bundle']  # type: Bundle
{% endhighlight %}
<p>We can use the transfer bundle tail transaction hash data to link a request using the create_request method. This method has a hot swappable url argument for API requests. For now it serves weather data based on city choice<\p>
{% highlight python %}
def create_request(url="http://46.101.109.238/forecast/", city='Chicago', headers=None):
    # type: (Text, Optional[dict]) -> dict
    return {"request": requests.get(url+city, headers=headers), "request-id": register_request()}
{% endhighlight %}

<p>So finally after the transactions ae attached to the tangle post payment the data will be served and the POC is complete!! As I said earlier there are much more powerful displays of the technology in the pipeline and we hope to show that this technology will revolutionize the IoT industry.<\p>

## Hello, Location Service!

#### This episode: Amazon Location Service and geospatial location services. In this Hello, Cloud blog series, we're covering the basics of AWS cloud services for newcomers who are .NET developers. If you love C# but are new to AWS, or to this particular service, this should give you a jumpstart.

In this post we'll introduce Amazon Location Service and use it in a "Hello, Cloud" .NET program for geocoding and route calculation, and also in a web page for interactive maps. We'll do this step-by-step, making no assumptions other than familiarity with C# and Visual Studio. We're using Visual Studio 2022 and .NET 6.

# Amazon Location Service : What is it, and why use It?

> “The three most important things about real estate are location, location, location.” – Harold Samuel

For the first half of my life, going somewhere new meant getting directions or consulting a printed map. If I was out somewhere, no person or system had an automatic way of knowing where I was or reaching me. Today, we're all very used to our location being known and never being lost, thanks to the devices we carry and GPS. We take location services for granted, such as maps, place name & address search, geocoding, and route calculation. How can you add location services to your own applications?

[Amazon Location Service](https://aws.amazon.com/location/) (hereafter "Location Service") is a service that allows you to integrate geospatial data and location functionality with your applications, including maps and geocoding. AWS describes it as "a service that makes it easy for developers to add location functionality, such as maps, points of interest, geocoding, routing, tracking, and geofencing, to their applications without sacrificing data security and user privacy." 

The primary features of Location Service are maps, places, routes, trackers, and geofencing. Many of these features let you select from multiple geospatial data providers, including ESRI and HERE. 

![diagram_features.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660472561776/NNliMfaX0.png align="left")

[Maps](https://docs.aws.amazon.com/location/latest/developerguide/map-concepts.html) visualize location information, are available in a variety of styles, and can be interleaved with your own data. 

![diagram_maps.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660473871365/CGVVX9Dpd.png align="left")

[Places](https://docs.aws.amazon.com/location/latest/developerguide/places-concepts.html) allows point-of-interest searches, and conversion to complete locations with coordinates (geocoding and reverse geocoding). 

![diagram_places.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660481239785/tTaWdPTe0.png align="left")

[Routes](https://docs.aws.amazon.com/location/latest/developerguide/route-concepts.html) calculate distance, travel time, and directions between two or more places.  

![diagram_routes.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660481537696/XDBpS9rhv.png align="left")

[Trackers](https://docs.aws.amazon.com/location/latest/developerguide/geofence-tracker-concepts.html) track current and historical locations of devices. Geofencing allows you detect and act when a device enters or exits a geographic boundary (geofence).

![diagram_tracking_geofencing.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660481701859/2_3ZkURbw.png align="left")

Location Service is a good fit for these use cases:
* Address enhancement
* Asset tracking
* Optimizing delivery routes
* Visualizing data on a map

The AWS Free Tier provides, for the first 3 months of usage, 500K map tiles, 10k addresses geocoded, 10k routes calculated, and more. Location Service charges vary by region, so be sure to consult the [pricing page](https://aws.amazon.com/location/pricing/). 

# Our Hello, Location Service Project

We'll write a .NET program that uses Location Service to search for and geocode places by name or address, perform "near" place searches, and calculate route distance and time between two places. We'll also embed a Location Service map in a web page.

![dotnet-run-search-address-nonspecific.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660430922874/n6qmbf_4A.png align="left")

![dotnet-run-search-name-pizza.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660433584101/KJu8UDkOr.png align="left")

![dotnet-run-route-usas.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660433624343/lF9_1RxJk.png align="left")

![map-page.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660430933481/oHTK_WJfJ.png align="left")

[source code](https://github.com/davidpallmann/hello-location)

## One-time Setup

For any of the tutorials in the Hello, Cloud series you need the following:

1. An  [AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/), and an understanding of what is included in the  [AWS Free Tier](https://aws.amazon.com/premiumsupport/knowledge-center/what-is-free-tier/). 
2. [Microsoft Visual Studio 2022](https://visualstudio.microsoft.com/). If you're using an older version of Visual Studio you won't be able to use .NET 6. If you use a different IDE, you may have to find alternatives to some tutorial steps.
3. [AWS Toolkit for Visual Studio](https://aws.amazon.com/visualstudio/). You'll need to [configure the toolkit ](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/keys-profiles-credentials.html) to access your AWS account and create an IAM user. Your default AWS profile will be linked to this user when running programs from the command line.

## Step 1: Configure Amazon Location Service

In this step, you'll create an Amazon Location Service **Place Index** you can search against. We'll be using the HERE provider.

1. Sign in to the Amazon management console and select the region you want to work in at the upper right. We're using us-west-2 (Oregon).

2. Navigate to **Location Service** and click **Create place index**.

    ![aws_create_place_index.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660416006957/1ffrJmS_y.png align="left")

    A. Name: **HelloLocationHERE**.

    B. Data provider: select **HERE**.

    C. Read the terms and conditions and check the box to acknowledge.

    D. Click **Create place index**.

    ![aws_create_place_index_HERE.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660416255386/p9y4QKjSg.png align="left")

3. Create a route calculator:

    A. Select **Route calculators** from the left pane.

    B. Click **Create route calculator**.

    C. Name: **HelloLocationHERE**.

    D. Data provider: select **HERE**.

    E. Review and agree to the terms and conditions.

    F. Click **Create route calculator**.

![aws-create-route-calc.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660422451130/UdPTiXJ8m.png align="left")

## Step 2: Create a .NET Program

In this step, you'll create a .NET console program that lets you exercise different features of the location service.

1. Open a command/terminal window and CD to a development folder.

2. Run the dotnet new command below create a new console program.

   ```dos
dotnet new console -n hello-location
```

3. Open the project in Visual Studio.

4. Add the **AWSSDK.Location** package to the project:

    A. In Solution Explorer, right-click the project and select **Manage NuGet Packages...**. 

    B. Search for and install the **AWSSDK.Location** package.

    ![nuget.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660414231688/HvIr7loRd.png align="left")

5. Open Program.cs in the code editor and replace with the code below. At the top of the code, set the `RegionEndpoint` to the region you are using, and confirm the Place Index and Route Calculator names match what you created in the AWS console.

6. Save your changes and ensure the program builds.

Program.cs

```csharp
using Amazon;
using Amazon.LocationService;
using Amazon.LocationService.Model;

namespace HelloLocation
{ 
    public class Program
    {
        static RegionEndpoint Region = RegionEndpoint.USWest2;
        static AmazonLocationServiceClient Client = null!;

        const string IndexName = "HelloLocationHERE";
        const string RouteCalculatorName = "HelloLocationHERE";

        static async Task Main(string[] args)
        {
            if (args.Length < 2 || args[0] == "-h")
            {
                Console.WriteLine("Usage:   dotnet run -- <action>");
                Console.WriteLine("Actions: search \"<place> ................ searches for places matching the text");
                Console.WriteLine("Actions: near \"<place1> <place2> ........ searches for place-1 near place-2");
                Console.WriteLine("Actions: route \"<place1> <place2> ....... finds the distance and route between place-1 and place-2");
                Console.WriteLine("Example: dotnet run -- geocode \"367 Wildwood Rd, Ronkonkoma, NY\"");
                Environment.Exit(1);
            }

            Client = new AmazonLocationServiceClient(Region);

            switch (args[0].ToLower())
            {
                case "search":

                    // "search" <place> - find place

                    var searchResponse = await SearchPlace(args[1]);
                    foreach (var result in searchResponse.Results)
                    {
                        Console.WriteLine($"{result.Place.Label} ({result.Place.Geometry.Point[0]}° , {result.Place.Geometry.Point[1]}°)");
                    }
                    break;

                case "near":

                    // "near" <place1> <place2> - find place1 near place2

                    var searchPlace2Response = await SearchPlace(args[2]);
                    if (searchPlace2Response.Results.Count == 0)
                    {
                        Console.WriteLine($"I couldn't find this place: {args[2]}");
                        Environment.Exit(2);
                    }

                    Console.WriteLine($"Searching for '{args[1]}' near {args[2]}");

                    // search for place1, using place2 as a bias position

                    var searchNearRequest = new SearchPlaceIndexForTextRequest()
                    {
                        IndexName = IndexName,
                        Text = args[1],
                        BiasPosition = searchPlace2Response.Results[0].Place.Geometry.Point
                    };

                    var searchNearResponse = await Client.SearchPlaceIndexForTextAsync(searchNearRequest);
                    foreach (var result in searchNearResponse.Results)
                    {
                        Console.WriteLine($"{result.Place.Label} ({result.Place.Geometry.Point[0]}° , {result.Place.Geometry.Point[1]}°)");
                    }
                    break;

                case "route":

                    // "route" <place1> <place2> - find the distance and duration between place 1 and place 2

                    var place1Response = await SearchPlace(args[1]);
                    if (place1Response.Results.Count == 0)
                    {
                        Console.WriteLine($"I couldn't find this place: {args[1]}");
                        Environment.Exit(2);
                    }
                    var place1 = place1Response.Results[0].Place;

                    var place2Response = await SearchPlace(args[2]);
                    if (place2Response.Results.Count == 0)
                    {
                        Console.WriteLine($"I couldn't find this place: {args[2]}");
                        Environment.Exit(2);
                    }
                    var place2 = place2Response.Results[0].Place;

                    var routeResponse = await GetRoute(place1, place2);

                    Console.WriteLine($"{place1.Label}\nis {routeResponse.Summary.Distance:F2} {routeResponse.Summary.DistanceUnit} from\n{place2.Label}");
                    Console.WriteLine($"Travel time: {routeResponse.Summary.DurationSeconds / 60:F2} minutes");
                    break;

                default:
                    Console.WriteLine("Unrecognized action");
                    break;
            }
        }

        /// <summary>
        /// Search for a place.
        /// </summary>
        /// <param name="text">Place name or address</param>
        /// <returns>SearchPlaceIndexForTextResponse</returns>

        static async Task<SearchPlaceIndexForTextResponse> SearchPlace(string text)
        {
            var location1Request = new SearchPlaceIndexForTextRequest()
            {
                IndexName = IndexName,
                Text = text
            };
            return await Client.SearchPlaceIndexForTextAsync(location1Request);
        }


        /// <summary>
        /// Calculate route between 2 places.
        /// </summary>
        /// <param name="place1">First place</param>
        /// <param name="place2">Second place</param>
        /// <returns>CalculateRouteResult</returns>

        static async Task<CalculateRouteResponse> GetRoute(Place place1, Place place2)
        {
            var routeRequest = new CalculateRouteRequest()
            {
                CalculatorName = RouteCalculatorName,
                DeparturePosition = place1.Geometry.Point,
                DestinationPosition = place2.Geometry.Point,
                DistanceUnit = DistanceUnit.Miles,
                TravelMode = TravelMode.Car
            };
            return await Client.CalculateRouteAsync(routeRequest);
        }
    }
}
```

### Understand the Code

This console program supports 3 actions: search, near, and route. 

| command | description |
| --- | ---- |
| dotnet run -- -h | display help |
| dotnet run -- search "place" | search for places matching the text |
| dotnet run -- near "place1" "place2" | search for place1 near place2 |
| dotnet run -- route "place1" "place2" | calculate the route from place1 to place2 |

At the top of Program.cs we set our region and the name of our  Place Index and Route Calculator.

**Command line processing** (lines 17-29). The command line expects the first parameter (`args[0]`) to be the action name. A place is specified in `args[1]` and, if a second place is expected, `args[2]`. A switch statement executes code based on the action specified.

**Search action** (31-40): This action searches the place name or address you supply, which can be partial, and returns matches. The local function `SearchPlace` is invoked, which calls the `SearchPlaceIndexForTextAsync` SDK method. The request specifies our Place Index and the place text to search. This can be a place name or address, full or partial. The calling code in `Main` loops through the response's `Results` property to display each `Place` object's label (full place name/address) and coordinates.

**Near action** (42-69). The near action is similar to search, but specifies a bias location for the search. To get the bias location, we first call `SearchPlace` on place2, because we will need to pass its coordinates to the search for place1. We then do the search for place1, specifying the first place2 result's `Place.Geometry.Point` property to specify the location to search near. 

**Route action** (71-95). The route action calculates the distance and travel time between two places. We call `SearchPlace` for place1 and for place2. Then we call our local method `GetRoute`, which calls the SDK's `CalculateRouteAsync` method. The request passes the point location of the departure and destination location. From the response, the calling code in `Main` displays the distance, distance unit of measurement, and travel time from the`Summary` property. Note that the request to `CalculateRouteAsync` can specify the mode of travel and unit of measurement for distance.

## Step 3: Test Geocoding

In this step, you'll test the .NET console program.

1. In a command/terminal window, CD to the project folder.

2. Test the **search** action. This action simply takes a place name, which could be an address or a name, exact or partial, and returns a list of places.

    A. Enter `dotnet run -- search ` followed by a specific, complete address. Remember to put the address in quotes. I'm testing an address from my childhood.

   ```dos
dotnet run -- search "367 Wildwood Rd, Ronkonkoma, NY"
```

    If you used a specific address, you'll get just one result. If you get no results, check for accuracy and spelling. If you see multiple results, your address wasn't specific enough. 

    ![dotnet-run-search-address.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660419415904/tDdrbBEaW.png align="left")

    B. Run the command again, and this time use a very non-specific address, such as **100 Main St**. 

   ```dos
dotnet run -- search "100 main st"
```

    You should see multiple results. 100 main st gives us results from around the world, including Gibraltar, Canada, and the US. For each results, we are displaying from the results the `Place` object's `Label` (geocoded name/address) and `Point` (coordinates) properties.

    ![dotnet-run-search-address-nonspecific.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660419559295/il-G97ph_.png align="left")

    C. Run the command again, and this time use a place name instead of an address. I'm using **Little Vincents Pizza**, our favorite family pizza place when I was growing up.

   ```dos
dotnet run -- search "Little Vincents Pizza"
```

    I'm happy to see my favorite pizzeria is still around, and has expanded to 3 locations.

   ![dotnet-run-search-name-pizza.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660419723586/zpl0z1TF7.png align="left")

3. Test the **near** action. This action is similar to **search**, but here you specify 2 places: the place you're looking for, and a place it is near. 

    A. Enter `dotnet run -- near ` followed by 2 parameters: **"100 main st"** and then **"New Orleans"**. We did this non-specific search earlier, but this time we're doing it with the qualifier "near New Orleans"

   ```dos
dotnet run -- near "100 main st" "New Orleans"
```

    The results are quite different from earlier. All but one of the addresses are in or near Louisiana. Some of the results in our earlier test of "100 main st" were left out, and more Louisiana results are listed here than before.

    ![dotnet-run-near-100main-nola.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660420333623/shYqGO8JB.png align="left")

    B. Enter `dotnet run -- near "Paris" "Dallas"` to search for Paris, but near Dallas.

   ```dos
dotnet run -- near "Paris" "Dallas"
```

   The results are nicely ordered by proximity to Dallas TX. The first result is on Paris St in the city of Dallas. The second is the city of Paris, TX. The third is Paris France.

    ![dotnet-run-near-paris-dallas.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660420648443/Hxa7wCCV8.png align="left")

4. Test the **route** action. 

    A. Enter the command below to get the distance between the Eiffel Tower and the Arc de Triomphe monument in Paris.

   ```dos
dotnet run -- route "eiffel tower" "arc de triomphe"
```

   You get a distance of 1.5 miles and a 6.15 minute duration.

    ![dotnet-run-route-paris.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660423505315/m274hUai6.png align="left")

    B. Run the command below to get the distance from the Golden Gate Bridge in San Francisco, California to the Brooklyn Bridge in New York.

    ```dos
dotnet run -- route "Golden Gate Bridge" "Brooklyn Bridge"
```

    ![dotnet-run-route-usas.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660423652714/n4jCV9a3Z.png align="left")

    C. Try the route action with your own places.

Congratulations! You've successfully geocoded places with Amazon Location Service in .NET code.

## Step 4: Test Embedded Maps in a Web Page

This step is optional and requires Node.js on your local computer. If you've come this far, you probably want to see the Location Service's map capability. In this step, you create a map in the AWS console, create a Cognito identity pool for authentication, and embed the map in a web page, backed up Node.js.

1. Create a map in the AWS console:

    A. Select **Maps** from the left pane.

    B. Click **Create map**.

    C. Name: **HelloLocationHERE**.

    D. Data provider: select **HERE Explore**.

    E. Review and agree to the terms and conditions.

    F. Click **Create map**.

    ![aws-create-map.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660425846878/GcWVK7ORK.png align="left")

    ![aws-create-map-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660425931040/pBzV1GDE2.png align="left")

2. Set up authentication for embedding the map. There are a lot of instructions presented in the console, you need to follow them carefully and completely.

    A. Click **Embed map**.

    B. Expand the **Set up authentication section**.

    C. Follow the instructions under **To create a new Amazon identity pool for use with this Amazon Location map** and perform all the steps, including adding an inline policy for the IAM role that is created.
 
    ![aws-create-map-auth-instructions.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660475683218/CgxCvufkh.png align="left")

    ![aws-create-map-auth-cognito.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660426229901/mOnvE2joS.png align="left")

    ![aws-create-map-auth-cognito-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660426426068/STHpOr8xI.png align="left")

    D. Record your identity pool ID and the code snipped displayed under Sample Code - Get AWS Credential.

    ![aws-create-map-auth-cognito-3.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660426790427/zugWX9Omr.png align="left")

    E. Expand the **Embed this map in your web application** section, and copy the HTML into a local file named index.html. In the `Auth` block, replace `REPLACE_WITH_POOL_ID`with the identity pool ID you just noted.
3. Launch the web page

    A. In a command/terminal window, CD to the project folder.

    B. Run the command `npx serve`. If the command is unknown, [install Node.js](https://nodejs.org/en/download/), open a new command window, and retry.

   ```dos
npx serve
```

    ![npx-serve.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660427328289/kJ545wume.png align="left")

    C. Note the URL displayed in the npx window, and browse to it. You should see a map.

    ![map-page.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660427482073/2rXOZt9Yj.png align="left")

    D. Interact with the map. Zoom in and out. Navigate to your home town.

    ![map-dallas.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660427937443/ud--vWHm0.png align="left")

    E. Stop `npx serve` from running in the command window.

Congratulations! You've successfully embedded an Amazon Location Services map in a web page.

## Step 5: Change Your Configuration and Experiment

This step is optional. You can change configuration to try different providers and map views.

1. Try an ESRI Place Index.

    A. Create another Place Index in the AWS console, as you did in Step 1, this time with **ESRI** as a provider. Name it HelloLocationESRI. 

    B. In your project, change the PlaceIndex constant in Program.cs to the new name.

    C. Build and run the program. Repeat the `search` and `near` action runs in Step 3, and note how the ESRI provider results differ from the HERE provider.

2. Try an ESRI Route Calculator.

    A. Create another Route Calculator in the AWS console, as you did in Step 1, this time with **ESRI** as a provider. Name it HelloLocationESRI. 

    B. In your project, change the RouteCalculator constant in Program.cs to the new name.

    C. Build and run the program. Repeat the `route` action runs in Step 3, and note how the ESRI provider results differ from the HERE provider.

3. Try a different map view

    A. Create a new Map in the console, as you did in Step 4-1, but choose a different map view this time.

    B. Select Embed Map and create an Index2.html on your local computer to reference the new map name.

    C. Edit the IAM policy you added in Step 4 and replace or add the new map resource name.

    D. In your command/terminal window, again run `npx serve` to run the web site, as you did in Step 4-3.

    E. Browse to the URL displayed by npx serve, and specify /Index2.html at the end of the path.

    F. How does the experience change with the new map?

    ![map2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660476341425/gnV4WsxTe.png align="left")

## Step 6: Shut it Down

When you're completed with the Hello, Location Service project, delete its resources. You won't want to accrue charges for something you're not using.

1. In the AWS console, navigate to **Location Service** and delete each of the Map, Route, and Place Index resources.

2. Navigate to **Cognito** and delete the identity pool.

# Where to Go From Here

Location services are becoming ubiquitous, and that means you likely have a need to provide them in your applications. Amazon Location Service provides maps, place search, routes, trackers, and geofencing capabilities. Many of these can be leveraged on the back end with the AWS SDK for .NET. Maps can be embedded in web site front ends.

In this tutorial, you created a Place Index and a Route Calculator in the AWS consoie. You wrote a .NET console program that performed place search, place search near another place, and route calculation. You also created a Map in the AWS console. You configured Cognito authentication and IAM role permissions for the map. You embedded the map in a local web page, served it with npx serve, and interacted with the map. 

We did not go deeply into route calculation to retrieve route details, adding your own data to maps, or trackers and geofencing. To go further, you'll want to delve deep into understanding the features you have use cases for. As always, practice building things to gain familiarity and proficiency with Amazon Location Service. Take advance of the AWS free tier.

# Further Reading

AWS Documentation

[Amazon Location Service](https://aws.amazon.com/location/)

[Developer Guide](https://docs.aws.amazon.com/location/latest/developerguide/welcome.html)

[AWS SDK for .NET Documentation](https://docs.aws.amazon.com/sdk-for-net/index.html)

Videos

[Amazon Location Service - Getting Started Videos](https://aws.amazon.com/location/getting-started/?location-whats-new.sort-by=item.additionalFields.postDateTime&location-whats-new.sort-order=desc&location-blogs.sort-by=item.additionalFields.createdDate&location-blogs.sort-order=desc)

Blogs

[Getting started with Amazon Location](https://aws.amazon.com/blogs/mobile/getting-started-with-amazon-location/) by Shu Sia Lukito

[Enriching addresses with AWS Lambda and the Amazon Location Service]() by James Beswick

[Amazon Location - Add Maps and Location Awareness to Your Applications](https://aws.amazon.com/blogs/aws/amazon-location-add-maps-and-location-awareness-to-your-applications/) by Jeff Barr

[Resources to Integrate with Amazon Location Service](https://aws.amazon.com/blogs/mobile/resources-to-integrate-with-amazon-location-service/) by Drishti Arora and Panna Shetty

[Add a map to your webpage with Amazon Location Service](https://aws.amazon.com/blogs/mobile/add-a-map-to-your-webpage-with-amazon-location-service/) by Kyle Lee

[Add Maps to your App in 3 Steps with AWS Amplify Geo powered by Amazon Location Service](https://aws.amazon.com/blogs/mobile/add-maps-to-your-app-in-3-steps-with-aws-amplify-geo/) by Harshita Daddala

[Add Interactive Maps in React using Amplify Geo, powered by Amazon Location Service](https://aws.amazon.com/blogs/mobile/add-interactive-maps-in-react-using-amplify-geo/)

[Hello, Cloud blog series](https://davidpallmann.hashnode.dev/series/hello-cloud)
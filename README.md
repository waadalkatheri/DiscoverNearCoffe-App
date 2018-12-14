Setting up Project
---------------------------
![alt text](https://d.top4top.net/p_10789bmu12.png)
![alt text](https://c.top4top.net/p_1078ben271.png)
![alt text](https://e.top4top.net/p_10784bje43.png)


# Near By Search
Steps 5:
1. Setting up dependencies
2. Setting up permissions
3. Setting up view
4. Setting up networking with Retrofit
5. Cross check.




2.Setting up permissions
-----------------------------------
Android Manifest
````
     <uses-permission android:name="android.permission.INTERNET"/>
     <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
     <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
````
More on <a href="./app/src/main/AndroidManifest.xml">AndroidManifest</a>



3.Setting up view
-----------------------------------
Map
````
        <fragment xmlns:android="http://schemas.android.com/apk/res/android"
              xmlns:tools="http://schemas.android.com/tools"
              android:id="@+id/map"
              android:name="com.google.android.gms.maps.SupportMapFragment"
              android:layout_width="match_parent"
              android:layout_height="match_parent"/>
````
More on <a href="./app/src/main/res/layout/activity_search_near_by_places.xml">Activity view</a>
https://ibb.co/F6wn6Cf
https://ibb.co/g7F803z
https://ibb.co/X3LTbBq
4.Setting up networking with Retrofit
-----------------------------------

Api service provider
````
    public NearByApi getApiService() {
        if (nearByApi == null) {
            HttpLoggingInterceptor interceptor = new HttpLoggingInterceptor();
            interceptor.setLevel(HttpLoggingInterceptor.Level.BODY);
            OkHttpClient client = new OkHttpClient.Builder().retryOnConnectionFailure(true).readTimeout(80, TimeUnit.SECONDS).connectTimeout(80, TimeUnit.SECONDS).addInterceptor(interceptor).build();
            
            Retrofit retrofit = new Retrofit.Builder().baseUrl(Constant.PLACE_API_BASE_URL).addConverterFactory(getApiConvertorFactory()).client(client).build();
            
            nearByApi = retrofit.create(NearByApi.class);
            return nearByApi;
        } else {
            return nearByApi;
        }
    }
    
    private static GsonConverterFactory getApiConvertorFactory() {
        return GsonConverterFactory.create();
    }
    
    
    public static MyApplication getApp() {
        return app;
    }
````
More on <a href="./app/src/main/java/com/parthdave/nearbysearch/MyApplication.java">Application Class</a>

Api call for finding places:
````
    public void findPlaces(String placeType){
            Call<NearByApiResponse> call = MyApplication.getApp().getApiService().getNearbyPlaces(placeType, location.getLatitude() + "," + location.getLongitude(), PROXIMITY_RADIUS);
        
            call.enqueue(new Callback<NearByApiResponse>() {
                @Override
                public void onResponse(Call<NearByApiResponse> call, Response<NearByApiResponse> response) {
                    try {
                        googleMap.clear();
                        // This loop will go through all the results and add marker on each location.
                        for (int i = 0; i < response.body().getResults().size(); i++) {
                            Double lat = response.body().getResults().get(i).getGeometry().getLocation().getLat();
                            Double lng = response.body().getResults().get(i).getGeometry().getLocation().getLng();
                            String placeName = response.body().getResults().get(i).getName();
                            String vicinity = response.body().getResults().get(i).getVicinity();
                            MarkerOptions markerOptions = new MarkerOptions();
                            LatLng latLng = new LatLng(lat, lng);
                            // Position of Marker on Map
                            markerOptions.position(latLng);
                            // Adding Title to the Marker
                            markerOptions.title(placeName + " : " + vicinity);
                            // Adding Marker to the Camera.
                            Marker m = googleMap.addMarker(markerOptions);
                            // Adding colour to the marker
                            markerOptions.icon(BitmapDescriptorFactory.defaultMarker(BitmapDescriptorFactory.HUE_RED));
                            // move map camera
                            googleMap.moveCamera(CameraUpdateFactory.newLatLng(latLng));
                            googleMap.animateCamera(CameraUpdateFactory.zoomTo(11));
                        }
                    } catch (Exception e) {
                        Log.d("onResponse", "There is an error");
                        e.printStackTrace();
                    }
                }
        
                @Override
                public void onFailure(Call<NearByApiResponse> call, Throwable t) {
                    Log.d("onFailure", t.toString());
                    t.printStackTrace();
                    PROXIMITY_RADIUS += 10000;
                }
            });
        }
````



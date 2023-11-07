# 준비과정
## API 이용시 알아둘 것
- API 명세서를 활용하여 API 사용방법 익히기
- API 이용시 중요한 정보는 config 파일을 생성하여 깃허브에 올라가지 않도록 하기

## 인터넷 이용
- 깃허브의 file 폴더에 있는 network_security_config 파일을 res/xml 안에 넣기
- AndroidManifest.xml파일 수정
```xml
<application
    android:networkSecurityConfig="@xml/network_security_config"
    android:usesCleartextTraffic="true"
</application>
```

## API를 이용 시 로딩화면
```java
// onCreate 외부
Dialog dialog;

void showProgress(){ // 프로그레스바 보여주도록 하는 함수
    dialog = new Dialog(this);
    dialog.setContentView(new ProgressBar(this));
    dialog.getWindow().setBackgroundDrawable(new ColorDrawable(Color.TRANSPARENT));
    dialog.setCancelable(false);
    dialog.setCanceledOnTouchOutside(false);
    dialog.show();
}

void dismissProgress(){ // 프로그레스바 숨기는 함수
    dialog.dismiss();
}
```

## volley 라이브러리 이용
### gradle(앱 수준)파일 수정
```gradle
dependencies {
    implementation 'com.android.volley:volley:1.2.1'
}
```

### java파일에서 api 호출
- api를 이용하여 데이터 가공하는 것을 parsing이라 함.
```java
// 네트워크 통신을 위한 Volley 라이브러리 사용.
RequestQueue queue = Volley.newRequestQueue(MainActivity.this); // 먼저 들어온 사람이 먼저 처리되도록(queue)(FIFO, first in first out)

// Request 요청을 만들어야 한다.
JsonObjectRequest request = new JsonObjectRequest(
        Request.Method.GET,
        "https://jsonplaceholder.typicode.com/posts",
        null,
        new Response.Listener<JSONObject>() {
            @Override
            public void onResponse(JSONObject response) {
                // 제이슨 오브젝트는 []로 감싸진 제이슨 어레이 안에 {}로 감싸진 제이슨코드

                // 1. 데이터를 가져오는 작업.
                // 네트워크로부터 데이터를 파싱한 후 어레이리스트에 추가!
                try {
                    for(int i=0; i<response.length(); i++) {
                        JSONObject object = response.getJSONObject(i);

                        int userId = object.getInt("userId");
                        int id = object.getInt("id");
                        String title = object.getString("title");
                        String body = object.getString("body");

                        Post post = new Post(userId, id, title, body);
                        postArrayList.add(post);


                    }

                } catch (JSONException e) {
                    return;
                }

                // 화면에 보여준다.
                adapter = new PostAdapter(MainActivity.this, postArrayList);
                recyclerView.setAdapter(adapter);
            }
        },
        new Response.ErrorListener() {
            @Override
            public void onErrorResponse(VolleyError error) {

            }
        }
);
queue.add(request1);
```

## retrofit 라이브러리 이용
### gradle(앱 수준)파일 수정
```gradle
dependencies {
    implementation 'com.squareup.retrofit2:retrofit:2.9.0'
    implementation 'com.squareup.retrofit2:converter-gson:2.9.0'
    implementation 'com.squareup.okhttp3:logging-interceptor:4.9.0'
}
```
### api 패키지에 retrofit 사용을 위한 준비
- file폴더의 NetworkClient.java 파일을 api 패키지를 생성하여 안에 넣기.

### api 패키지에 인터페이스 생성
#### 예시1
```java
// 레트로핏 API 사용 메뉴얼에 따라서
// 인터페이스로 만든다.
public interface MemoApi {

    // 내 메모 리스트 가져오는 API
    @GET("/memo")
    Call<MemoList> getMemoList(@Header("Authorization") String token);

    // 메모 생성하는 API
    @POST("/memo")
    Call<ResultRes> addMemo(@Header("Authorization") String token, @Body Memo memo);

    // 메모 수정하는 API
    @PUT("/memo/{memoId}")
    Call<ResultRes> updateMemo(@Path("memoId") int memoId, @Header("Authorization") String token, @Body Memo memo);

    // 메모 삭제하는 API
    @DELETE("/memo/{memoId}")
    Call<ResultRes> deleteMemo(@Path("memoId") int memoId, @Header("Authorization") String token);
}
```

#### 예시2
```java
public interface PostApi {

    // 친구들의 포스트 가져오는 API
    @GET("/post/follow")
    Call<PostList> getFollowPost(@Query("offset") int offset,
                                 @Query("limit") int limit,
                                 @Header("Authorization") String token);

    // 포스트 좋아요 누르기
    @POST("/post/{postId}/like")
    Call<ResultRes> setPostLike(@Path("postId") int postId,
                                @Header("Authorization") String token);

    // 포스트 좋아요 취소
    @DELETE("/post/{postId}/like")
    Call<ResultRes> deletePostLike(@Path("postId") int postId,
                                   @Header("Authorization") String token);

    //포스트 생성 API
    @Multipart
    @POST("/post")
    Call<ResultRes> addPost(@Header("Authorization") String token,
                            @Part MultipartBody.Part photo,
                            @Part("content") RequestBody content);
}
```

#### 예시3
```java
public interface UserApi {

    // 회원가입 API
    @POST("/user/register")
    Call<UserRes> register(@Body User user);

    // 로그인
    @POST("user/login")
    Call<UserRes> login(@Body User user);
}
```

### retrofit 이용
```java
// 네트워크로 데이터를 보낸다.
Retrofit retrofit = NetworkClient.getRetrofitClient(AddActivity.this);

MemoApi api = retrofit.create(MemoApi.class);

Memo memo = new Memo(title, date + " " + time, content);

Call<ResultRes> call = api.addMemo("Bearer " + token, memo);

showProgress();

call.enqueue(new Callback<ResultRes>() {
    @Override
    public void onResponse(Call<ResultRes> call, Response<ResultRes> response) {
        dismissProgress();

        if(response.isSuccessful()) {

            finish();
        } else {
            Snackbar.make(btnSave,
                    "서버와 통신이 되지 않습니다.",
                    Snackbar.LENGTH_SHORT).show();
        }
    }

    @Override
    public void onFailure(Call<ResultRes> call, Throwable t) {
        dismissProgress();
        
    }
});
```

# 유튜브 API
```java
private void getNetworkData() {
    progressBar.setVisibility(View.VISIBLE);

    youtubeArrayList.clear();

    RequestQueue queue = Volley.newRequestQueue(MainActivity.this);

    JsonObjectRequest request = new JsonObjectRequest(
            Request.Method.GET,
            Config.HOST + Config.PATH + "?key=" + Config.GCP_API_KEY + "&part=snippet&maxResults=20&type=video&q=" + keyword,
//                        "https://www.googleapis.com/youtube/v3/search?part=snippet&key=" + 키 + "&q=" + keyword + "&maxResults=20&type=video",
            null,
            new Response.Listener<JSONObject>() {
                @Override
                public void onResponse(JSONObject response) {
                    try {
                        pageToken = response.getString("nextPageToken");

                        JSONArray items = response.getJSONArray("items");

                        for (int i = 0; i < items.length(); i++) {

                            JSONObject JsonObject = items.getJSONObject(i);
                            String videoId = JsonObject.getJSONObject("id").getString("videoId");

                            JSONObject snippet = JsonObject.getJSONObject("snippet");
                            String title = snippet.getString("title");
                            String description = snippet.getString("description");
                            String url = snippet.getJSONObject("thumbnails").getJSONObject("medium").getString("url");

                            Youtube youtube = new Youtube(videoId, title, description, url);
                            youtubeArrayList.add(youtube);

                        }


                    } catch (JSONException e) {

                        Snackbar.make(imgSearch,
                                "데이터 파싱 에러",
                                Snackbar.LENGTH_SHORT).show();
                        return;
                    }

                    adapter = new YoutubeAdapter(MainActivity.this, youtubeArrayList);
                    progressBar.setVisibility(View.GONE);
                    recyclerView.setAdapter(adapter);

                }
            },
            new Response.ErrorListener() {
                @Override
                public void onErrorResponse(VolleyError error) {
                    progressBar.setVisibility(View.GONE);

                }
            }
    );

    queue.add(request);
}
```

# 파파고 API
- 라디오버튼을 활용한 API 호출
- Volley 라이브러리 이용시 헤더에 데이터 셋팅방법

```java
// 1. 어떤 언어로 번역할지에 대한 정보를 가져온다.
// 눌려진 라디오 버튼 정보를 가져온다.
String target;

int radioButtonId = radioGroup.getCheckedRadioButtonId();

if(radioButtonId == R.id.radioButton1){
    target = "en";
}else if(radioButtonId == R.id.radioButton2){
    target = "zh-CN";
}else if(radioButtonId == R.id.radioButton3){
    target = "zh-TW";
}else if(radioButtonId == R.id.radioButton4){
    target = "th";
}else{
    Snackbar.make(button,
            "번역할 언어를 선택하세요.꼭!",
            Snackbar.LENGTH_SHORT).show();
    return;
}

// 2. 에디트텍스트에 유저가 작성한 글을 가져온다.
String text = editText.getText().toString().trim();
if(text.isEmpty()){
    Snackbar.make(button,
            "번역할 문장을 입력하세요 꼭!",
            Snackbar.LENGTH_SHORT).show();
    return;
}

// 3. 파파고 API를 호출한다. 그 결과를 화면에 보여준다.
String source = "ko";

RequestQueue queue = Volley.newRequestQueue(MainActivity.this);

String url = "https://openapi.naver.com/v1/papago/n2mt";

JSONObject body = new JSONObject();
try {
    body.put("source", source);
    body.put("target", target);
    body.put("text", text);

} catch (JSONException e) {
    return;
}

JsonObjectRequest request = new JsonObjectRequest(
        Request.Method.POST,
        url,
        body, // 포스트는 보내야 하는 정보가 있기에 null이 아님.
        new Response.Listener<JSONObject>() {
            @Override
            public void onResponse(JSONObject response) {
                // 네이버로부터 받은 데이터를 처리하고 화면에 표시

                try {
                    String result = response.getJSONObject("message").getJSONObject("result").getString("translatedText");
                    txtResult.setText(result);
                    Papago papago = new Papago(text, target, result);
                    papagoArrayList.add(0, papago);

                } catch (JSONException e) {
                    return;
                }


            }
        },
        new Response.ErrorListener() {
            @Override
            public void onErrorResponse(VolleyError error) {

            }
        }

){
    // 발리에서 헤더에 데이터를 셋팅하고 싶으면 이렇게 한다.
    @Override
    public Map<String, String> getHeaders() throws AuthFailureError {
        Map<String, String> header = new HashMap<>();

        header.put("X-Naver-Client-Id", "네이버 클라이언트 아이디"); // config 파일에 문자열을 저장하여 사용하기
        header.put("X-Naver-Client-Secret", "네이버 클라이언트 시크릿");

        return header;
    }
};
queue.add(request);
```
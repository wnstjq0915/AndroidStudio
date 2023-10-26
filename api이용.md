# 인터넷 이용
- 깃허브의 file 폴더에 있는 network_security_config 파일을 res/xml 안에 넣기
- AndroidManifest.xml파일 수정
```xml
<application
    android:networkSecurityConfig="@xml/network_security_config"
    android:usesCleartextTraffic="true"
</application>
```

## volley 라이브러리 이용
- gradle(앱 수준)파일 수정
```gradle
dependencies {
    implementation 'com.android.volley:volley:1.2.1'
}
```

- java파일애서 api 호출
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
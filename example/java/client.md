# Client Code

``` java
package org.cresplanex.api.state.userprofileservice.service;

import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.util.Map;
import java.util.concurrent.Executors;
import java.util.logging.Level;
import java.util.logging.Logger;

import com.google.common.util.concurrent.ListenableFuture;
import com.google.common.util.concurrent.ListeningExecutorService;
import com.google.common.util.concurrent.MoreExecutors;

import build.buf.gen.user.v1.GetUserRequest;
import build.buf.gen.user.v1.GetUserResponse;
import build.buf.gen.user.v1.UserServiceGrpc;
import build.buf.gen.user.v1.UserServiceGrpc.UserServiceStub;
import io.grpc.Channel;
import io.grpc.ManagedChannel;
import io.grpc.ManagedChannelBuilder;
import io.grpc.MethodDescriptor;
import io.grpc.stub.StreamObserver;

public class UserService {

    /**
     * ---------------------------------------------------------
     *
     * Client-side code
     *
     * --------------------------------------------------------
     */
    public Channel createChannel() {
        ManagedChannel channel = ManagedChannelBuilder
                .forAddress("localhost", 8080) // サーバーのアドレスとポートを指定
                .usePlaintext() // 開発環境ではTLSを使用せず平文通信（本番環境ではTLS推奨）
                .build();

        return channel;
    }

    /**
     * 簡易リクエスト作成
     *
     * @return GetUserRequest リクエスト
     */
    public GetUserRequest createRequest() {
        return GetUserRequest.newBuilder().setUserId("user1").build();
    }

    /**
     * 非同期スタブを使用してリクエストを送信
     *
     * @param chanel チャネル
     * @param request リクエスト
     */
    public void asyncStub(Channel channel, GetUserRequest request) {
        UserServiceStub asyncStub = UserServiceGrpc.newStub(channel);
        asyncStub.getUser(request, new StreamObserver<GetUserResponse>() {
            @Override
            public void onNext(GetUserResponse response) {
                System.out.println("Received async response: " + response.getName());
            }

            @Override
            public void onError(Throwable t) {
                System.err.println("Error in async call: " + t.getMessage());
            }

            @Override
            public void onCompleted() {
                System.out.println("Async call completed");
            }
        });
    }

    /**
     * 同期スタブを使用してリクエストを送信
     *
     * @param channel チャネル
     * @param request リクエスト
     */
    public void blockingStub(Channel channel, GetUserRequest request) {
        UserServiceGrpc.UserServiceBlockingStub blockingStub = UserServiceGrpc.newBlockingStub(channel);
        GetUserResponse response = blockingStub.getUser(request);
        System.out.println("Received blocking response: " + response.getName());
    }

    /**
     * フューチャースタブを使用してリクエストを送信
     *
     * @param channel チャネル
     * @param request リクエスト
     */
    public void futureStub(Channel channel, GetUserRequest request) {
        UserServiceGrpc.UserServiceFutureStub futureStub = UserServiceGrpc.newFutureStub(channel);
        ListeningExecutorService pool = MoreExecutors.listeningDecorator(Executors.newFixedThreadPool(10));

        ListenableFuture<GetUserResponse> future = futureStub.getUser(request);
        future.addListener(() -> {
            System.out.println("Received future response[1]: " + future.toString());
        }, pool);

        future.addListener(() -> {
            System.out.println("Received future response[2]: " + future.toString());
        }, pool);

        // future.addListener(() -> {
        //     System.out.println("Received future response[2]: " + future.toString());
        // }, MoreExecutors.directExecutor());
    }
    
    /**
     * ---------------------------------------------------------
     *
     * Metadata
     *
     * --------------------------------------------------------
     */
    /**
     * サービス名を取得
     *
     * @return
     */
    public String getServiceName() {
        return UserServiceGrpc.SERVICE_NAME;
    }

    /**
     * メソッドメタデータを取得
     *
     * @return
     */
    public MethodDescriptor<GetUserRequest, GetUserResponse> getGetUserMethod() {
        return UserServiceGrpc.getGetUserMethod();
    }

    /**
     * メソッド情報を取得
     *
     * @return
     */
    public Map<String, Object> getMethods() {
        Map<String, Object> methods = Map.of();

        MethodDescriptor<GetUserRequest, GetUserResponse> ds = getGetUserMethod();

        methods.put("bareMethodName", ds.getBareMethodName()); // メソッド名
        methods.put("fullMethodName", ds.getFullMethodName()); // フルメソッド名
        methods.put("requestMarshaller", ds.getRequestMarshaller()); // リクエストマーシャラー
        methods.put("responseMarshaller", ds.getResponseMarshaller()); // レスポンスマーシャラー
        methods.put("type", ds.getType()); // タイプ(UNARY, CLIENT_STREAMING, SERVER_STREAMING, BIDI_STREAMING, UNKNOWN)
        methods.put("idempotent", ds.isIdempotent()); // べき等であるか
        methods.put("safe", ds.isSafe()); // サーバー側への副作用がないか
        methods.put("sampledToLocalTracing", ds.isSampledToLocalTracing()); // ローカルトレースストアにサンプリングされるかどうか
        methods.put("schemaDescriptor", ds.getSchemaDescriptor()); // スキーマ記述子
        methods.put("service", ds.getServiceName()); // サービス名

        try (InputStream inputStream = new FileInputStream("request_in.txt")) {
            GetUserRequest request = ds.parseRequest(inputStream);
            methods.put("request", request); // リクエスト
        } catch (IOException e) {
            Logger.getLogger(UserService.class.getName()).log(Level.SEVERE, null, e);
        }

        try (InputStream inputStream = new FileInputStream("response_in.txt")) {
            GetUserResponse response = ds.parseResponse(inputStream);
            methods.put("response", response); // レスポンス
        } catch (IOException e) {
            Logger.getLogger(UserService.class.getName()).log(Level.SEVERE, null, e);
        }

        GetUserRequest request = GetUserRequest.newBuilder().setUserId("user1").build();
        InputStream req = ds.streamRequest(request);
        methods.put("streamRequest", req); // ストリームリクエスト

        GetUserResponse response = GetUserResponse.newBuilder().setUserId("user1").setEmail("test@example.com").build();
        InputStream res = ds.streamResponse(response);
        methods.put("streamResponse", res); // ストリームレスポンス

        return methods;
    }
}

```
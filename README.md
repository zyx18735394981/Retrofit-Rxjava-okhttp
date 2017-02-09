# Retrofit-Rxjava-okhttp
     Retrofit+Rxjava+okhttp总结
        1.Retrofit+Rxjava+okhttp基本使用方法
        2.统一处理请求数据格式
        3.统一的ProgressDialog和回调Subscriber处理
        4.取消http请求
        5.预处理http请求
        6.返回数据的统一判断
        7.失败后的retry封装处理
        8.RxLifecycle管理生命周期，防止泄露 
        
        具体使用

    封装后http请求代码如下
        //    完美封装简化版
            private void simpleDo() {
                SubjectPost postEntity = new SubjectPost(simpleOnNextListener,this);
                postEntity.setAll(true);
                HttpManager manager = HttpManager.getInstance();
                manager.doHttpDeal(postEntity);
            }

            //   回调一一对应
            HttpOnNextListener simpleOnNextListener = new HttpOnNextListener<List<Subject>>() {
                @Override
                public void onNext(List<Subject> subjects) {
                    tvMsg.setText("已封装：\n" + subjects.toString());
                }

                /*用户主动调用，默认是不需要覆写该方法*/
                @Override
                public void onError(Throwable e) {
                    super.onError(e);
                    tvMsg.setText("失败：\n" + e.toString());
                }
            };

        是不是很简单？你可能说这还简单，好咱们对比一下正常使用Retrofit的方法

        /**  
            * Retrofit加入rxjava实现http请求  
            */  
           private void onButton9Click() {  
               //手动创建一个OkHttpClient并设置超时时间  
               okhttp3.OkHttpClient.Builder builder = new OkHttpClient.Builder();  
               builder.connectTimeout(5, TimeUnit.SECONDS);  

               Retrofit retrofit = new Retrofit.Builder()  
                       .client(builder.build())  
                       .addConverterFactory(GsonConverterFactory.create())  
                       .addCallAdapterFactory(RxJavaCallAdapterFactory.create())  
                       .baseUrl(HttpManager.BASE_URL)  
                       .build();  

        /        加载框  
               final ProgressDialog pd = new ProgressDialog(this);  

               HttpService apiService = retrofit.create(HttpService.class);  
               Observable<RetrofitEntity> observable = apiService.getAllVedioBy(true);  
               observable.subscribeOn(Schedulers.io()).unsubscribeOn(Schedulers.io()).observeOn(AndroidSchedulers.mainThread())  
                       .subscribe(  
                               new Subscriber<RetrofitEntity>() {  
                                   @Override  
                                   public void onCompleted() {  
                                       if (pd != null && pd.isShowing()) {  
                                           pd.dismiss();  
                                       }  
                                   }  

                                   @Override  
                                   public void onError(Throwable e) {  
                                       if (pd != null && pd.isShowing()) {  
                                           pd.dismiss();  
                                       }  
                                   }  

                                   @Override  
                                   public void onNext(RetrofitEntity retrofitEntity) {  
                                       tvMsg.setText("无封装：\n" + retrofitEntity.getData().toString());  
                                   }  

                                   @Override  
                                   public void onStart() {  
                                       super.onStart();  
                                       pd.show();  
                                   }  
                               }  

                       );  
           }

     如果你一个activity或者fragment中多次需要http请求，你需要多次重复的写回调处理（一个回到就有4个方法呀！！！！反正我是忍受不了），而且以上处理还没有做过多的判断和错误校验就如此复杂！
     要继续优化！！！！
     --------------参考RxJava+Retrofit+OkHttp深入浅出-终极封装二（网络请求）

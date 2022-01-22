1. TFM调用IVR彩虹桥外呼接口
2. IVR彩虹桥调用SIPCS外呼名单系统
3. SIPCS调用EPM soap接口发起外呼请求
4. epm接受到请求执行ccxml
5. ccxml拨打客户手机
6. 客户接听后，转接到IVR流程进行操作，同时回调TFM接口
7. 客户拒绝，回调TFM接口

异常：epm soap接口返回拒绝信息
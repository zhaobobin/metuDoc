# 模型 Model



#### global.js

```js
import { routerRedux } from 'dva/router';
import { notification } from 'antd';
import Request from '@/utils/request';
import Storage from '@/utils/storage';
import ENV from '@/config/env'

export default {

  namespace: 'global',

  state: {

    loading: true,

    isAuth: false,
    status: undefined,
    lastTel: Storage.get(ENV.storageLastTel) || '',
    currentUser: '',                                  //当前用户信息
    roleList: [],                                     //角色列表

    signModalVisible: false,                          //登录modal的显示状态
    signTabKey: '',                                   //登录modal中tab的默认key

    theme: Storage.get(ENV.storageTheme) || {},
  },

  subscriptions: {
    setup({ dispatch, history }) {  // eslint-disable-line

    },
  },

  effects: {
    *init({ payload }, { call, put }) {
      const res = yield call(
        (params) => {return Request('/api/init', {method: 'POST', body: params})},
        payload
      );
      if(res.status === 1){
        yield put({
          type: 'changeAppInfo',
          payload: res.data,
        });
      }
    },
    *register({ payload, callback }, { call, put }) {
      const res = yield call(
        (params) => {return Request('/api/register', {method: 'POST', body: params})},
        payload
      );
      if(res.status === 1){
        yield put({
          type: 'changeLoginStatus',
          payload: {
            loading: false,
            isAuth: true,
            currentUser: res.data.currentUser,
          }
        });
        Storage.set(ENV.storageLastTel, payload.tel);
        Storage.set(ENV.storageToken, res.data.token);              //保存token
      }
      yield callback(res);
    },

    *login({ payload, callback }, { call, put }) {
      const res = yield call(
        (params) => {return Request('/api/login', {method: 'POST', body: params})},
        payload
      );
      if(res.status === 1){
        yield put({
          type: 'changeLoginStatus',
          payload: {
            loading: false,
            isAuth: true,
            currentUser: res.data.currentUser,
          }
        });
        Storage.set(ENV.storageLastTel, payload.tel);
        Storage.set(ENV.storageToken, res.data.token);              //保存token
      }
      yield callback(res);
    },

    *token({ payload, callback }, { call, put }) {

      //没有refreshToken 不校验token接口
      if(payload.refreshToken) {

        const res = yield call(
          (params) => {return Request('/api/token', {method: 'POST', body: params})},
          payload
        );
        yield callback(res);

        if(res.status === 1){
          Storage.set(ENV.storageToken, res.data.token);               //保存token
          yield put({
            type: 'changeLoginStatus',
            payload: {
              loading: false,
              isAuth: true,
              currentUser: res.data,
            }
          });
        }else{
          yield put({
            type: 'changeLoginStatus',
            payload: {
              loading: false,
              isAuth: false,
              currentUser: '',
            }
          });
        }

      }else{
        yield put({ type: 'changeLoading', payload: false });
      }

    },

    *logout({ payload, callback }, { call, put }) {
      const res = yield call(
        (params) => {return Request('/api/logout', {method: 'POST', body: params})},
        payload
      );
      if(res.status === 1){
        yield put({
          type: 'changeLoginStatus',
          payload: {
            isAuth: false,
            currentUser: '',
          },
        });
        Storage.remove(ENV.storageToken);
      }
    },

    *post({ url, payload, callback }, { call, put }) {

      let res, exp = payload.exp, storage = Storage.get(url);

      if(exp && storage){
        res = storage;
      }else{
        res = yield call(
          (params) => {return Request(url, {method: 'POST', body: params})},
          payload
        );
        if(res.status === 1 && exp) Storage.set(url, res);
      }

      //登录过期等
      if(res.status === 9){
        Storage.remove(ENV.storageToken);              //删除token
        notification.error({
          message: '提示',
          description: res.message
        });
        yield put({
          type: 'changeLoginStatus',
          payload: {
            isAuth: false,
            userInfo: '',
          }
        });
        yield put(routerRedux.push({ pathname: '/' }));
      }else{
        yield callback(res);
      }

    },

    *get({ url, payload, callback }, { call, put }) {
      const res = yield call(
        (params) => {return Request(url, {method: 'GET', body: params})},
        payload
      );
      yield callback(res);
    },
  },

  reducers: {
    changeAppInfo(state, { payload }) {
      return {
        ...state,
        loading: false,
        isAuth: !!payload.currentUser,
        roleList: payload.roleList,
        category: payload.category,
        currentUser: payload.currentUser,
      };
    },
    changeLoginStatus(state, { payload }) {
      return {
        ...state,
        loading: payload.loading,
        isAuth: payload.isAuth,
        currentUser: payload.currentUser,
        token: payload.token,
      };
    },
    changeSignModal(state, { payload }) {
      return {
        ...state,
        signModalVisible: payload.signModalVisible,
        signTabKey: payload.signTabKey,
      };
    },
  },

};


```



#### oss.js

```js
/**
 * oss 文件上传
 key生成规则：用户id/保存类别_当前时间戳.图片类型
 let key = this.props.global.currentUser._id + '/photo_' + new Date().getTime() + '.' + file.name.split('.')[1];
 */
import request, {FetchGet} from "@/utils/request"
import {notification} from 'antd'

const getClient = function (data) {
  return new window.OSS.Wrapper({
    region: data.region,
    bucket: data.bucket,
    accessKeyId: data.credentials.AccessKeyId,
    accessKeySecret: data.credentials.AccessKeySecret,
    stsToken: data.credentials.SecurityToken
  })
}
//base64转Blob
const toBlob = function (urlData, fileType) {
  let bytes = window.atob(urlData),
    n = bytes.length,
    u8arr = new Uint8Array(n)
  while (n--) {
    u8arr[n] = bytes.charCodeAt(n)
  }
  return new Blob([u8arr], {type: fileType})
}

export default {
  namespace: 'oss',

  state: {
    loading: false,
    submitting: false,
    exif: ''
  },

  effects: {

    // 列表
    *list({payload, callback}, {call, put}) {
      yield put({
        type: 'changeLoading',
        payload: true,
      })
      const res = yield call(
        (params) => {
          return request('/api/PhotoList', {method: 'POST', body: params,})
        },
        {payload}
      )
      yield callback(res)
      yield put({
        type: 'changeList',
        payload: res,
      })
      yield put({
        type: 'changeLoading',
        payload: false,
      })
    },

    // 详情
    *detail({payload, callback}, {call, put}) {
      yield put({
        type: 'changeLoading',
        payload: true,
      })
      const res = yield call(
        (params) => {
          return request('/api/FileOssEdit', {method: 'POST', body: params,})
        },
        {}
      )
      yield callback(res)
      yield put({
        type: 'changeDetail',
        payload: res,
      })
      yield put({
        type: 'changeLoading',
        payload: false,
      })
    },

    // 源数据
    *exif({payload, callback}, {call, put}) {

      let url = payload.url + '?x-oss-process=image/info';

      yield FetchGet(url)
      .then((exif) => {
        if (exif.FNumber) {
          let FNumber = exif.FNumber.value                              //转换光圈值
          exif.FNumber.value = (FNumber.split('/')[0] / FNumber.split('/')[1]).toString()
        }
        return callback(exif)
      })
      .catch(function (err) {
        notification.error({
          message: 'exif查询失败！',
          description: err,
        })
      })
    },

    // 上传
    *upload({payload, callback}, {call, put}) {
      const res = yield call(
        (params) => {
          return request('/api/FileOssEdit', {method: 'POST', body: params,})
        },
        {}
      )
      if (res.status === 1) {
        let client = getClient(res.data)
        client.multipartUpload(payload.key, payload.file, {})
          .then(function (r) {
            let url = r.res.requestUrls[0].split('?')[0]
            url = url.replace('metuwang.oss-cn-qingdao.aliyuncs.com', 'photo.metuwang.com')
            return callback(url)
          })
          .catch(function (err) {
            notification.error({
              message: '上传失败！',
              //description: err,
            })
          })
      }
    },

    *uploadBase64({payload, callback}, {call, put}) {
      const res = yield call(
        (params) => {
          return request('/api/FileOssEdit', {method: 'POST', body: params,})
        },
        {}
      )
      if (res.status === 1) {

        let client = getClient(res.data)

        let dataUrl = payload.base64
        let base64 = dataUrl.split(',')[1]
        let fileType = dataUrl.split(';')[0].split(':')[1]
        let blob = toBlob(base64, fileType)
        let reader = new FileReader()

        reader.readAsArrayBuffer(blob)
        reader.onload = function (event) {
          // arrayBuffer转Buffer
          let buffer = new window.OSS.Buffer(event.target.result)
          client.put(payload.key, buffer).then(function (r) {
            let url = r.res.requestUrls[0].split('?')[0]
            url = url.replace('metuwang.oss-cn-qingdao.aliyuncs.com', 'photo.metuwang.com')
            return callback(url)
          }).catch(function (err) {
            notification.error({
              message: '上传失败！',
              description: err,
            })
          })
        }
      }

    },

    *del({payload, callback}, {call, put}) {
      yield put({
        type: 'changeSubmitting',
        payload: true,
      })
      const res = yield call(
        (params) => {
          return request('/api/FileOssEdit', {method: 'POST', body: params,})
        },
        {}
      )
      if (res.status === 1) {
        let client = getClient(res.data)
        client.deleteMulti(payload.keys)
          .catch(function (err) {
            notification.error({
              message: '删除失败！',
              description: err,
            })
          })
      }
      yield put({
        type: 'changeSubmitting',
        payload: false,
      })
    },

  },

  reducers: {
    changeLoading(state, {payload}) {
      return {
        ...state,
        loading: payload,
        uploadAvatar: '',
      }
    },
    changeSubmitting(state, {payload}) {
      return {
        ...state,
        submitting: payload,
      }
    },
  }

}

```



#### publish.js

```js
/**
 * 内容发布 - 用于保存编辑文章的状态
 */
import Request from '@/utils/request';

export default {

  namespace: 'publish',

  state: {

    submitting: false,                    //提交状态
    publishType: '',                      //发布类型：文章、图片
    content: '',                          //发布内容
    thumb: '',                            //缩略图（可选）
  },

  effects: {
    *init({ payload }, { call, put }) {
      const res = yield call(
        (params) => {return Request('api/init', {method: 'POST', body: params})},
        payload
      );
      if(res.status === 1){
        yield put({
          type: 'changeAppInfo',
          payload: res,
        });
      }
    },

  },

  reducers: {
    changePublishType(state, {payload}) {
      return {
        ...state,
        publishType: payload.publishType
      };
    },
    saveArticleContent(state, {payload}) {
      return {
        ...state,
        content: payload.content,
        thumb: payload.thumb,
      };
    },
    savePhotoContent(state, {payload}) {
      return {
        ...state,
        content: payload.content,
        thumb: payload.thumb,
      };
    },
  }
}

```


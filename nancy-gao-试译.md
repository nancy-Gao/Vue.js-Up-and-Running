# 第六章 Vuex的状态管理

本书到此为止，所有的数据都被存储在我们的组件中。我们调用 API ，将返回的数据存储在数据对象中。我们将一个表单和一个对象进行绑定，并将这个对象存储在一个数据对象中。组件之间所有的通信使用事件（从子组件到父组件）和 props （从父组件到子组件）来完成。这对简单场景来说是友好的，但是在更加复杂的应用程序中，这并不够。

让我们拿一个社交应用程序举例，具体来说，就是消息。你想要一个图标在导航栏顶部展示你的消息数量，你想要一个消息弹出页面的底部，这时也会告诉你的消息数量。这两个组件在页面上相距遥远，所以使用事件和 props 连接他们将会是一场噩梦：组件完全和通知无关，但却必须知道传递到他们的事件。另一种方法是，使用共享数据连接组件来替代以上做法，你可以从每个组件单独发出 API 请求。这将会更糟糕！因为每个组件会在不同的时间点更新，这意味着他们会展示不同的内容，页面需要发起超出它原本所需的 API 请求数量。

__vuex__ 是一个帮助开发者在 Vue 应用程序管理他们应用程序状态的方法库。它提供了一个集中式存储，你可以在整个应用程序存储和处理全局状态，并使你可以验证数据，确保数据出现也是可预测和正确的。

## 安装

你可以通过 CDN 使用 Vuex，只需要添加如下语句：

    <script src="https://unpkg.com/vuex"></script>

此外，如果你正在使用npm，你可以使用 __npm install --save vuex__ 安装 vuex 。如果你正在使用打包工具，例如 webpack ，那么正如使用 vue-router ，你需要调用 Vue.use()：

    import Vue from 'vue';
    import Vuex from 'vuex';

    Vue.use(Vuex);

然后你需要建立你的存储。让我们建立如下一个文件，并将它存储为 store/index.js：

    import Vuex from 'vuex';

    export default new Vuex.Store({
      state: {}
    });

目前为止，那只是一个空的存储：但我们将在整个章节中添加内容给它。

再之后，在你的主应用程序文件中引入它，在创建 Vue 实例时，让它作为一个属性加入。

    import Vue from 'vue';
    import store from './store';

    new Vue({
      el: '#app',
      store,
      components: {
        App
      }
    });

现在你已经把存储加入到你的应用程序，你可以通过 __this.$store__ 获取到它。下面让我们来看看 Vuex 的概念，以及你能通过 __this.$store__ 做什么。

## 概念

正如本章在介绍中提到的，当复杂的应用程序需要多个组件共享状态，那么 Vuex 就可以上场了。

让我们不用 Vuex 写一个简单的组件，这个组件在页面上展示一个用户的消息数量:

    const NotificationCount = {
      template: `<p>Messages: {{ messageCount }}</p>`,
      data: () => ({
        messageCount: 'loading'
      }),
      mounted() {
        const ws = new WebSocket('/api/messages');
        ws.addEventListener('message', (e) => {
          const data = JSON.parse(e.data);
          this.messageCount = data.messages.length;
        });
      }
    };

这是非常简单的。它打开了一个 __/api/message__ 的 websocket，之后当服务器发送数据给客户端————在这个例子中，当套接字（socket）被打开（初始化消息计数），当计数被更新（有新消息）时————消息被发送到套接字，计数，并展示到页面上。

![3-1]

在实践中，这段代码将更复杂：在这个例子中， websocket 没有认证，并且它总是假定 websocket 是由有效的 JSON 格式响应，带有数组属性的消息，但现实可能不会。对于这个示例，简单的代码将完成这项工作。

当我们在一个页面使用多个通知计数组件的时候，我们就会遇到问题。当每一个组件打开一个 websocket 的时候，它会开启不必要的重复连接。并且因为网络延迟，组件会在略微差异的时间点更新。为了修复这个问题，我们将 websocket 的逻辑放进 vuex。

让我们通过一个例子，深入理解。我们的组件会变成如下：

    const NotificationCount = {
      template: `<p>Messages: {{ messageCount }}</p>`, computed: {
        messageCount() {
          return this.$store.state.messages.length;
        }
      }
      mounted() {
        this.$store.dispatch('getMessages');
      }
  };

然后如下是我们的 vuex 存储：

[3-1]:/images-nancy-gao/3-1.png

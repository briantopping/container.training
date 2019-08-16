# Declarative vs imperative

Kubernetes is *declarative*

- **Declarative:** Please provide what I describe

  *I would like a cup of tea.*

- **Imperative:** Please do these specific tasks in order

  *Boil some water. Pour it in a teapot. Add tea leaves. Steep for a while. Serve in a cup.*

--

Both result in a cup of tea!

---

# Declarative vs imperative

- Declarative is "opinionated". If a declarative system "adds leaves before pouring the water", some users might have no end of problems (some of which are non-technical in nature).

--

- Imperative is exhausting and potentially error prone if every user has to describe the same basic tasks as every other user.

--

- Upgrading imperative *ecosystems* is difficult:
  - If users of a system have specified where they want a cup of tea, new versions of the system might use filtered water automatically. *When would someone not want filtered water for their tea?*
  
  - Harder to do this if users have specified imperative rules in each deployment.

---

## Declarative vs imperative

- Imperative systems:

  - Higher user confidence because we are controlling the outcome

  - An abstract system controller doen't know what's going on. Have to restart from scratch.
    - *Can the system unwind the imperative side effects?*

- Declarative systems:

  - if a process is interrupted (or if we show up to the party half-way through), we can figure out what's missing and do only what's necessary

  - the *controller* must track the current *state* from it's own actions

  - ... and compute a "diff" between that *state* and desired *specification*

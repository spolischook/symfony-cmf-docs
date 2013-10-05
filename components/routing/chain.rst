.. index::
    single: Routing; Components; ChainRouter

ChainRouter
===========

At the core of the Symfony CMF Routing component sits the ``ChainRouter``. It
is used as a replacement for Symfony2's default routing system.

The ``ChainRouter`` works by accepting a set of prioritized routing
strategies, :class:`Symfony\\Component\\Routing\\RouterInterface`
implementations, commonly referred to as "routers".

When handling an incoming request, the ``ChainRouter`` iterates over the
configured routers, sorted by their configured priority, until one of them is
able to :method:`match <Symfony\\Component\\Routing\\Matcher\\RequestMatcherInterface::matchRequest>`
the request or to :method:`match <Symfony\\Component\\Routing\\RouterInterface::match>`
the URL and provide the request parameters.

.. note::

    Historically, the router had to do the matching on the URL string alone.
    Since Symfony 2.2, it can alternatively implement the
    ``RequestMatcherInterface`` to do the matching on the full
    :class:`Symfony\\Component\\Routing\\Request` object if it wants to use
    other request information into account like the domain or accept-encoding.
    The ``ChainRouter`` supports both types of route matching.

Adding Routers to the Chain
---------------------------

Routers are added using the ``add`` method of the ``ChainRouter``. Use this to
add the default Symfony2 router::

    use Symfony\Component\Routing\Router;
    use Symfony\Cmf\Component\Routing\ChainRouter;

    $chainRouter = new ChainRouter();
    $chainRouter->add(new Router(...));
    $chainRouter->match('/foo/bar');

Now, when the ``ChainRouter`` matches a request, it will ask the Symfony2
``Router`` to match see if it matches. If there is no match, it will throw a
:class:`Symfony\\Component\\Routing\\Exception\\ResourceNotFoundException`.

If you add a new router, for instance the ``DynamicRouter``, it will be
called after the Symfony2 Router (because that was added first). To control the
order, you can use the second argument of the ``add`` method to set a priority.
Higher priorities are sorted first.

.. code-block:: php

    use Symfony\Cmf\Component\Routing\DynamicRouter;
    // ...

    $chainRouter->add(new Router(...), 1);

    $dynamicRouter = new DynamicRouter(...);
    // ...
    $chainRouter->add($dynamicRouter, 100);

.. note::

    You'll learn how to instantiate the :doc:`DynamicRouter <dynamic>`
    later in this article.

Router Compiler Pass
~~~~~~~~~~~~~~~~~~~~

This component provides a ``RegisterRoutersPass``. If you use the Symfony2
Dependency Injection Container, you can use this compiler pass to register all
routers with a specific tag::

    use Symfony\Cmf\Component\Routing\DependencyInjection\Compiler\RegisterRoutersPass;
    use Symfony\Component\DependencyInjection\ContainerBuilder;

    $container = ... // ContainerBuilder
    $pass = new RegisterRoutersPass();
    $pass->process($container);

The defalt tag is ``router``. If you are using the Symfony2 CMF RoutingBundle,
this tag is already active with the default name.

Routers
-------

The ``ChainRouter`` is incapable of, by itself, making any actual routing
decisions. Its sole responsibility is managing the given set of Routers,
which are responsible for matching a request and determining its parameters.

You can easily create your own routers by implementing
:class:`Symfony\\Component\\Routing\\RouterInterface` but the Symfony CMF
Routing Component already includes a powerful route matching system that you
can extend to your needs.

Symfony2 Default Router
~~~~~~~~~~~~~~~~~~~~~~~

The Symfony2 routing mechanism is itself a ``RouterInterface`` implementation,
which means you can use it as a Router in the ``ChainRouter``. This allows you
to use the default routing declaration system. Read more about this router in
the `Routing Component`_ article of the core documentation.

Dynamic Router
~~~~~~~~~~~~~~

The dynamic router is best added with a lower priority, as the default router
is faster in taking routing decisions.

Read on about the dynamic router in the :doc:`next section<dynamic>`.

.. _`Routing Component`: http://symfony.com/doc/current/components/routing/introduction.html

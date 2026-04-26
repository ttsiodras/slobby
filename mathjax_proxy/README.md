This is a proxy for [slobby](https://github.com/itkach/slobby); it extracts the math definitions from the "alt" field of non-inline math images, and uses Mathjax to render them.

Before, with just slobby:

![Before, with just slobby](images/before.png "Before, with just slobby")

After, over this mathjax_proxy.py:

![After, over this mathjax_proxy.py](images/after.png "After, over this mathjax_proxy.py")

Usage is simple:

    python3 -m venv .venv
    . .venv/bin/activate
    python3 -m pip install -r requirements.txt
    python3 mathjax_proxy.py

By default, it proxies the traffic from http://127.0.0.1:8014 to the "backend" slobby, which defaults to port 8013.

Hope it helps someone.

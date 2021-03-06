# pdf-set

Provide PModel for set of PDF (probability density function) for [range coder](https://github.com/diegodox/range_coder_rust)

## installation

`Cargo.toml`

```toml
[dependencies]
# this crate provides range_coder's PModel.
pdf_set = {git="https://github.com/diegodox/pdf-set.git"}
# also we need rangecoder provides Encoder/Decoder
range_coder = {git="https://github.com/diegodox/range_coder_rust.git", branch="carryless"}
```
## example

```rust
struct GaussianDist {
    h: f64,
    w: f64,
    m: u8,
}
impl PDF for GaussianDist {
    fn freq(&self, v: usize) -> f64 {
        fn delta(a: usize, b: usize) -> usize {
            if a > b { a - b } else { b - a }
        }
        let t = -1f64 * self.w.powi(2) * (delta(v , self.m as usize) as f64).powi(2);
        self.h
            * self.w
            * t.exp()
    }
}
fn test() {
    let ansewr = vec![34, 45, 128, 255, 0];
    
    // PModel
    // these are PDFs
    let g1 = GaussianDist {
        h: std::f64::MAX,
        w: std::f64::MIN_POSITIVE,
        m: 128,
    };
    let g2 = GaussianDist {
        h: 10.0,
        w: 2.0,
        m: 30,
    };
    let g3 = GaussianDist {
        h: 2.0,
        w: 5.0,
        m: 70,
    };
    let set = PDFSet::new(vec![g1, g2, g3]);
    let pm = set.finalize();
    
    // encoding
    let mut encoder = Encoder::new();
    for i in &ansewr {
        encoder.encode(&pm, *i);
    }
    encoder.finish();
    // encoded bytes
    let data = encoder.data().clone();
    
    // decoding
    let mut decoder = Decoder::new();
    decoder.set_data(data);
    decoder.decode_start();
    let mut decoded = Vec::new();
    for _ in 0..ansewr.len() {
        decoded.push(decoder.decode_one_alphabet(&pm));
    }
    
    // check
    assert_eq!(ansewr, decoded);
}
```

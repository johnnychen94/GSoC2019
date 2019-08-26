# GSoC Final Report

This is the summary report of my GSoC 2019 project: [Towards Better Images Ecosystem](https://summerofcode.withgoogle.com/projects/#4861272753963008)

## Summary

There're many packages maintained or developed by me during this period:

* [`ImageNoise.jl`](https://github.com/johnnychen94/ImageNoise.jl)
* [`ImageDistances.jl`](https://github.com/JuliaImages/ImageDistances.jl)
* [`ImageQualityIndexes.jl`](https://github.com/JuliaImages/ImageQualityIndexes.jl)
* [`DemoCards.jl`](https://github.com/johnnychen94/DemoCards.jl)
* [`ImageBinarization.jl`](https://github.com/zygmuntszpak/ImageBinarization.jl)

There're some work I planned to do during the GSoC but it turns out to be a large project. For example,

* I got lost in the details of `ImageFiltering.jl` when I tried to introduce new APIs to it.
* I created some test utils during the development, but I don't have time to polish them and gather them into a `ImageTests.jl` package.

---

## Details

This section collects all the non-trivial PRs and commits I made during April 22 and Aug 26. Issues and PR Reviews are not included.

### 1. `ImageNoise.jl`

As is written in the proposal, [`ImageNoise.jl`](https://github.com/johnnychen94/ImageNoise.jl) is developed as a experimental ground to validate some of my ideas. It now supports

* `apply_noise` API with noise type: `AdditiveGaussianNoise`
* `reduce_noise` API with denoiser: `Non-local mean`

During the development of this package, I

* made the first version of the API style used in other packages
* discussed the general strategy of image processing with the community

After GSoC, there's a plan to support more noise types and denoise methods like [`ImageBinarization.jl`](https://github.com/zygmuntszpak/ImageBinarization.jl)

---

### 2. `ImageCore.jl`, `Colors.jl` and `FixedPointNumbers.jl`

[`ImageCore.jl`](https://github.com/JuliaImages/ImageCore.jl), [`Colors.jl`](https://github.com/JuliaGraphics/Colors.jl) and [`FixedPointNumbers.jl`](https://github.com/JuliaMath/FixedPointNumbers.jl) are core packages in JuliaImages ecosystem.

I added several new features to this package:

* [Feature] [enhance and export floattype](https://github.com/JuliaMath/FixedPointNumbers.jl/pull/122)
* [Feature] [extend `float` to `Colorant`](https://github.com/JuliaImages/ImageCore.jl/pull/87)
* [Feature] Add NumberLike and GrayImageLike traits in [#76](https://github.com/JuliaImages/ImageCore.jl/pull/76) and [#80](https://github.com/JuliaImages/ImageCore.jl/pull/80)
* [Maintainance] reduce direct dependencies of downstream packages on FixedPointNumbers and Colors in [reexport Colors and FixedPointNumbers](https://github.com/JuliaImages/ImageCore.jl/pull/75)
* [Bug] [Remove `N*f0` symbols](https://github.com/JuliaMath/FixedPointNumbers.jl/pull/121)
* [Feature] [add inplace operation clamp01! and clamp01nan](https://github.com/JuliaImages/ImageCore.jl/pull/81)
* [Enhancement] [enhance Colordiff API](https://github.com/JuliaGraphics/Colors.jl/pull/338) with broadcasting support

#### TODO List in the Future

- [ ] [poor performance of `N0f8`](https://github.com/JuliaMath/FixedPointNumbers.jl/issues/125)
- [ ] [rework `ColorVectorSpaces` in a more consistent way](https://github.com/JuliaGraphics/ColorVectorSpace.jl/issues/38)

---

### 3. `ImageDistances.jl`

`ImageDistances.jl` lacked of maintainance; it only partially supported `CIEDE2000` and `Hausdorff` distances. I took over the maintainance work of this from Júlio Hoffimann because he's busy with his daily work.

I gave new life to this package, some noteworthy features/PRs are:

* [Redesign the codebase](https://github.com/JuliaImages/ImageDistances.jl/pull/14) redesigns the type hierarchy used in ImagesDistances.jl, this enables us to reuse the codes in [Distances.jl](https://github.com/JuliaStats/Distances.jl)
* [Add Color support](https://github.com/JuliaImages/ImageDistances.jl/pull/19) support generic Color3 images, it also discusses the general rules on color operations.
* [Crosstype distances stability](https://github.com/JuliaImages/ImageDistances.jl/pull/23) supports inter-operation between `Gray` and `Number`, `Float32` and `N0f8`
* [support ndarray](https://github.com/JuliaImages/ImageDistances.jl/pull/26) support generic n-D array
* [rework dispatches and update compatibility](https://github.com/JuliaImages/ImageDistances.jl/pull/32) is the second attempt redesigning the code base. This makes the codes easier to understand and maintain.

There're some other PRs in the upstream packages that enables the development of `ImageDistances.jl`:

* [Enhancement] [rework result_type](https://github.com/JuliaStats/Distances.jl/pull/140) and [move implementations into type overloading (aka. functor)](https://github.com/JuliaStats/Distances.jl/pull/139) in [Distance.jl](https://github.com/JuliaStats/Distances.jl) enables the second redesign of `ImageDistances.jl` by splitting API and implementations apart.
* [Bug] [correct NormRMSDeviation type](https://github.com/JuliaStats/Distances.jl/pull/131) in [Distance.jl](https://github.com/JuliaStats/Distances.jl).

#### After the work

There're 11 new distances introduced in `ImageDistances.jl`:

* `SumAbsoluteDifference`
* `SumSquaredDifference`
* `MeanAbsoluteError`
* `MeanSquaredError`
* `RootMeanSquaredError`
* `SqEuclidean`
* `Euclidean`
* `Cityblock`
* `Minkowski`
* `Hamming`
* `TotalVariation`

All distances (unless explicitly noted) support input with different image types, while the output are still identical. For example, the following types are covered by test cases:

* `Array{Gray{N0f8}}`, `Array{Gray{Float32}}`, and `Array{Gray{Bool}}`
* `Array{N0f8}`, `Array{Float32}` and `Array{Bool}`
* `Array{RGB{N0f8}}` and `Array{RGB{Float32}}`

#### TODO List in the future

- [ ] [move diff-related functions to ImageDistances.jl](https://github.com/JuliaImages/ImageDistances.jl/issues/17)
- [ ] [Performance benchmark](https://github.com/JuliaImages/ImageDistances.jl/issues/27)
- [ ] Pre-v`0.3` documentation update
- [ ] reduce `ReferenceTest.jl` dependency on `Images.jl` to `ImageDistances.jl`

---

### 4. `ImageQualityIndexes.jl`

This is a *new* package that supports different image quality functions:

* [PSNR](https://github.com/JuliaImages/ImageQualityIndexes.jl/pull/2) (Peak Signal-to-Noise Ratio)
* [SSIM](https://github.com/JuliaImages/ImageQualityIndexes.jl/pull/3) (Structural similarity)
* [colorfulness](https://github.com/JuliaImages/ImageQualityIndexes.jl/pull/9) -- `HASLER_AND_SUSSTRUNK_M3`

During the developement of this package, I've found a developing pattern that can be adapted to other JuliaImages packages. The reasoning behind this is documented in [The principle of images -- part I](https://nextjournal.com/johnnychen94/the-principles-of-imagesjl-part-i).

There's no development plan on this package in the future except:

- [ ] reexport `ImageQualityIndexes.jl` by `Images.jl`

---

### 5. `ImageBinarization.jl`

After the first release of `ImageQualityIndexes.jl`, I refactored the codebase of [`ImageBinarization.jl`](https://github.com/zygmuntszpak/ImageBinarization.jl) using the same pattern I found in `ImageQualityIndexes.jl`.

All PRs are merged to a root PR [refactor codebase with functor APIs](https://github.com/zygmuntszpak/ImageBinarization.jl/pull/29) first, and when all things are settled down, this root PR is merged to master.

As a summary, this refactorization consists of:

* [Feature] refactor the codebase using the functor API discussed in [Filter specification: an alternative to `binarize`](https://github.com/zygmuntszpak/ImageBinarization.jl/issues/26)
* [Feature] add in-place function `binarize!`
* [Feature] support `Color3` inputs
* [Maintainance] add more test codes
* [Documentation] slightly enhance the documentation

#### TODO List in the future

- [ ] Performance Benchmark
- [ ] Refactor the upstream package [`HistogramThresholding.jl`](https://github.com/zygmuntszpak/HistogramThresholding.jl) and [`ImageContrastAdjustment.jl`](https://github.com/zygmuntszpak/ImageContrastAdjustment.jl) with the same approach.

### 6. Documentation

As I'm gradually understanding the overall design of the JuliaImages ecosystem, I think I'm qualified to update some easy part of the documentation:

* [Maintainance] [move dependency from travis to Project.toml](https://github.com/JuliaImages/juliaimages.github.io/pull/57)
* [Documentation] [Rewrite the quick start](https://github.com/JuliaImages/juliaimages.github.io/pull/58)

#### TODO list in the future

- [ ] The current documentation isn't very friendly to new users; there's a need to rewrite it.
- [ ] add more demos for new beginners

---

### 7. `DemoCards.jl`

_This package can benefit the whole Julia community._

When I was trying to update [JuliaImages documentation](https://juliaimages.org/latest/), I found it difficult for other contributors to add new demos to the documentation systems. To solve this issue, I developed a new package [`DemoCards.jl`](https://github.com/johnnychen94/DemoCards.jl) to simplify the pipeline of adding a new demo.

A usage demo of this package can be seen [here](https://johnnychen94.github.io/DemoCards.jl/dev/democards/gallery_of_packages/).

This package is written by me alone and not hosted under `JuliaImages`. To implement the ideas quickly, I didn't choose to develop it in form of PRs, instead, I directly pushes my commits to master.

#### TODO List in the future

- [ ] add more test cases to increase the coverage
- [ ] support URL cards
- [ ] support julia cards
- [ ] add post-processing to the pipeline

---

### 8. `Images.jl`

* [Bug] [add missing ImageMorphology import (fixes #796)](https://github.com/JuliaImages/Images.jl/pull/797)
* [Maintainance] [remove array operation `red`, `green`, `blue`](https://github.com/JuliaImages/Images.jl/pull/808)
* [Maintainance] port graph-related codes from Images.jl in [ImageMorphology #17](https://github.com/JuliaImages/ImageMorphology.jl/pull/17) and [Images.jl #809](https://github.com/JuliaImages/Images.jl/pull/809) -- This solves a blocking issue in a feature PR [add imfill function](https://github.com/JuliaImages/ImageMorphology.jl/pull/9)
* [Maintainance] [switching registration system](https://github.com/JuliaImages/Images.jl/issues/798) for almost all JuliaImages packages.

---

### 9. Miscellaneous

PRs listed here are made during the three-month development to repositories I don't have developer permission:

* [Bug] [Update Project.toml and move testutils to test/](https://github.com/JuliaStats/Distributions.jl/pull/866) in [`Distributions.jl`](https://github.com/JuliaStats/Distributions.jl)
* [Bug] [fix show for BenchmarkConfig with id being nothing](https://github.com/JuliaCI/PkgBenchmark.jl/pull/80) in [`PkgBenchmark.jl`](https://github.com/JuliaCI/PkgBenchmark.jl)
* [Bug] [correct the behavior of centered for OffsetArray](https://github.com/JuliaImages/ImageFiltering.jl/pull/104)
* [Bug] [Pending] [0 length image is not showable](https://github.com/JuliaImages/ImageShow.jl/pull/13) in [`ImageShow.jl`](https://github.com/JuliaImages/ImageShow.jl)
* [Bug] [Pending] [rework basename](https://github.com/JuliaLang/julia/pull/33021) in [julia](https://github.com/JuliaLang/julia)
* [Bug] [fix permission error](https://github.com/abelsiqueira/jill/pull/9) in [`jill`](https://github.com/abelsiqueira/jill)
* [Feature] [add MacOS support](https://github.com/abelsiqueira/jill/pull/8) in [`jill`](https://github.com/abelsiqueira/jill)


---

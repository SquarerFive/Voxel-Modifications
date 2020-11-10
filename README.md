# VoxelNodeFunctions.h

## Bicubic Interpolation for the Voxel Float Texture sample

### Implementation - Line 610
```cpp
inline FVector CubicHermite(FVector A, FVector B, FVector C, FVector D, float t)
	{
		float t2 = t * t;
		float t3 = t * t * t;
		FVector a = -A / 2.0f + (3.0f * B) / 2.0f - (3.0f * C) / 2.0f + D / 2.0f;
		FVector b = A - (5.0f * B) / 2.0f + 2.0f * C - D / 2.0f;
		FVector c = -A / 2.0f + C / 2.0f;
		FVector d = B;
		return a * t3 + b * t2 + c * t + d;
	}
	inline FVector SampleTexture(const TVoxelTexture<FColor>& Texture, FVector2D UV, const EVoxelSamplerMode Mode)
	{
		float R;
		float G;
		float B;
		//float A;
		const FLinearColor Color = Texture.Sample<FLinearColor>(UV.X, UV.Y, Mode);
		R = Color.R;
		G = Color.G;
		B = Color.B;
		return FVector(R, G, B);

	}
	
	inline FVector BicubicHermiteTextureSample(const TVoxelTexture<FColor>& Texture, FVector2D Pixel, const EVoxelSamplerMode Mode)
	{
		FVector2D TextureSize = FVector2D(Texture.GetSizeX(), Texture.GetSizeY());
		FVector2D frac_ = FVector2D(FMath::Frac(Pixel.X), FMath::Frac(Pixel.Y));
		FVector2D pixel = FVector2D(FMath::FloorToFloat(Pixel.X+0.5), FMath::FloorToFloat(Pixel.Y+0.5));

		//float maxP = FMath::Max(TextureSize.X, TextureSize.Y);
		// float onePixel = 1.0f / maxP;

		float onePixel = 50.0f;
		float twoPixels = 100.0f;
		FVector C00 = SampleTexture(Texture, pixel + FVector2D(-onePixel, -onePixel), Mode);
		FVector C10 = SampleTexture(Texture, pixel + FVector2D(0.0f, -onePixel), Mode);
		FVector C20 = SampleTexture(Texture, pixel + FVector2D(onePixel, -onePixel), Mode);
		FVector C30 = SampleTexture(Texture, pixel + FVector2D(twoPixels, -onePixel), Mode);

		FVector C01 = SampleTexture(Texture, pixel + FVector2D(-onePixel, 0.0f), Mode);
		FVector C11 = SampleTexture(Texture, pixel + FVector2D(0.0f, 0.0f), Mode);
		FVector C21 = SampleTexture(Texture, pixel + FVector2D(onePixel, 0.0f), Mode);
		FVector C31 = SampleTexture(Texture, pixel + FVector2D(twoPixels, 0.0f), Mode);

		FVector C02 = SampleTexture(Texture, pixel + FVector2D(-onePixel, onePixel), Mode);
		FVector C12 = SampleTexture(Texture, pixel + FVector2D(0.0f, onePixel), Mode);
		FVector C22 = SampleTexture(Texture, pixel + FVector2D(onePixel, onePixel), Mode);
		FVector C32 = SampleTexture(Texture, pixel + FVector2D(twoPixels, onePixel), Mode);

		FVector C03 = SampleTexture(Texture, pixel + FVector2D(-onePixel, twoPixels), Mode);
		FVector C13 = SampleTexture(Texture, pixel + FVector2D(0.0f, twoPixels), Mode);
		FVector C23 = SampleTexture(Texture, pixel + FVector2D(onePixel, twoPixels), Mode);
		FVector C33 = SampleTexture(Texture, pixel + FVector2D(twoPixels, twoPixels), Mode);
		
		FVector CP0X = CubicHermite(C00, C10, C20, C30, frac_.X);
		FVector CP1X = CubicHermite(C01, C11, C21, C31, frac_.X);
		FVector CP2X = CubicHermite(C02, C12, C22, C32, frac_.X);
		FVector CP3X = CubicHermite(C03, C13, C23, C33, frac_.X);

		return CubicHermite(CP0X, CP1X, CP2X, CP3X, frac_.Y);
		
	}
	inline FVector SampleTextureLinear(const TVoxelTexture<float>& Texture, FVector2D UV, const EVoxelSamplerMode Mode)
	{
		float R;
		float G;
		float B;
		//float A;
		const float Value = Texture.Sample<float>(UV.X, UV.Y, Mode);
		R = Value;
			G = Value;
			B = Value;
		return FVector(R, G, B);

	}
	inline FVector CubicLagrange(FVector A, FVector B, FVector C, FVector D, float t)
	{
		float c_x0 = -1.0;
		float c_x1 = 0.0;
		float c_x2 = 1.0;
		float c_x3 = 2.0;
		return
			FVector(
			A *
			(
				(t - c_x1) / (c_x0 - c_x1) *
				(t - c_x2) / (c_x0 - c_x2) *
				(t - c_x3) / (c_x0 - c_x3)
				) +
			B *
			(
				(t - c_x0) / (c_x1 - c_x0) *
				(t - c_x2) / (c_x1 - c_x2) *
				(t - c_x3) / (c_x1 - c_x3)
				) +
			C *
			(
				(t - c_x0) / (c_x2 - c_x0) *
				(t - c_x1) / (c_x2 - c_x1) *
				(t - c_x3) / (c_x2 - c_x3)
				) +
			D *
			(
				(t - c_x0) / (c_x3 - c_x0) *
				(t - c_x1) / (c_x3 - c_x1) *
				(t - c_x2) / (c_x3 - c_x2)
				));
	}
	inline FVector BicubicHermiteTextureSampleLinear(const TVoxelTexture<float>& Texture, FVector2D Pixel, const EVoxelSamplerMode Mode)
	{
		FVector2D TextureSize = FVector2D(Texture.GetSizeX(), Texture.GetSizeY());
		FVector2D pixel = Pixel + 0.5f;
		FVector2D frac_ = FVector2D(Fractional(pixel.X), Fractional(pixel.Y));
		
		v_flt c_onePixel = 1.f;
		v_flt c_twoPixels = 2.f;
		pixel = FVector2D(floorf(pixel.X), floorf(pixel.Y)) - FVector2D(c_onePixel / 2.0f);
		


		FVector C00 = SampleTextureLinear(Texture, pixel + FVector2D(-c_onePixel, -c_onePixel), Mode);
		FVector C10 = SampleTextureLinear(Texture, pixel + FVector2D(0.0, -c_onePixel), Mode);
		FVector C20 = SampleTextureLinear(Texture, pixel + FVector2D(c_onePixel, -c_onePixel), Mode);
		FVector C30 = SampleTextureLinear(Texture, pixel + FVector2D(c_twoPixels, -c_onePixel), Mode);
		FVector C01 = SampleTextureLinear(Texture, pixel + FVector2D(-c_onePixel, 0.0), Mode);
		FVector C11 = SampleTextureLinear(Texture, pixel + FVector2D(0.0, 0.0), Mode);
		FVector C21 = SampleTextureLinear(Texture, pixel + FVector2D(c_onePixel, 0.0), Mode);
		FVector C31 = SampleTextureLinear(Texture, pixel + FVector2D(c_twoPixels, 0.0), Mode);
		FVector C02 = SampleTextureLinear(Texture, pixel + FVector2D(-c_onePixel, c_onePixel), Mode);
		FVector C12 = SampleTextureLinear(Texture, pixel + FVector2D(0.0, c_onePixel), Mode);
		FVector C22 = SampleTextureLinear(Texture, pixel + FVector2D(c_onePixel, c_onePixel), Mode);
		FVector C32 = SampleTextureLinear(Texture, pixel + FVector2D(c_twoPixels, c_onePixel), Mode);
		FVector C03 = SampleTextureLinear(Texture, pixel + FVector2D(-c_onePixel, c_twoPixels), Mode);
		FVector C13 = SampleTextureLinear(Texture, pixel + FVector2D(0.0, c_twoPixels), Mode);
		FVector C23 = SampleTextureLinear(Texture, pixel + FVector2D(c_onePixel, c_twoPixels), Mode);
		FVector C33 = SampleTextureLinear(Texture, pixel + FVector2D(c_twoPixels, c_twoPixels), Mode);

		FVector CP0X = CubicHermite(C00, C10, C20, C30, frac_.X);
		FVector CP1X = CubicHermite(C01, C11, C21, C31, frac_.X);
		FVector CP2X = CubicHermite(C02, C12, C22, C32, frac_.X);
		FVector CP3X = CubicHermite(C03, C13, C23, C33, frac_.X);

		return CubicHermite(CP0X, CP1X, CP2X, CP3X, frac_.Y);

	}
	inline FVector BicubicLagrangeTextureSampleLinear(const TVoxelTexture<float>& Texture, FVector2D Pixel, const EVoxelSamplerMode Mode)
	{
		FVector2D TextureSize = FVector2D(Texture.GetSizeX(), Texture.GetSizeY());
		FVector2D pixel = Pixel + 0.5f;
		FVector2D frac_ = FVector2D(Fractional(pixel.X), Fractional(pixel.Y));
		
		v_flt c_onePixel = 1.f;
		v_flt c_twoPixels = 2.f;
		pixel = FVector2D(floorf(pixel.X), floorf(pixel.Y)) - FVector2D(c_onePixel/2.0f);
		

		
		FVector C00 = SampleTextureLinear(Texture, pixel + FVector2D(-c_onePixel, -c_onePixel), Mode);
		FVector C10 = SampleTextureLinear(Texture, pixel + FVector2D(0.0, -c_onePixel), Mode);
		FVector C20 = SampleTextureLinear(Texture, pixel + FVector2D(c_onePixel, -c_onePixel), Mode);
		FVector C30 = SampleTextureLinear(Texture, pixel + FVector2D(c_twoPixels, -c_onePixel), Mode);
		FVector C01 = SampleTextureLinear(Texture, pixel + FVector2D(-c_onePixel, 0.0), Mode);
		FVector C11 = SampleTextureLinear(Texture, pixel + FVector2D(0.0, 0.0), Mode);
		FVector C21 = SampleTextureLinear(Texture, pixel + FVector2D(c_onePixel, 0.0), Mode);
		FVector C31 = SampleTextureLinear(Texture, pixel + FVector2D(c_twoPixels, 0.0), Mode);
		FVector C02 = SampleTextureLinear(Texture, pixel + FVector2D(-c_onePixel, c_onePixel), Mode);
		FVector C12 = SampleTextureLinear(Texture, pixel + FVector2D(0.0, c_onePixel), Mode);
		FVector C22 = SampleTextureLinear(Texture, pixel + FVector2D(c_onePixel, c_onePixel), Mode);
		FVector C32 = SampleTextureLinear(Texture, pixel + FVector2D(c_twoPixels, c_onePixel), Mode);
		FVector C03 = SampleTextureLinear(Texture, pixel + FVector2D(-c_onePixel, c_twoPixels), Mode);
		FVector C13 = SampleTextureLinear(Texture, pixel + FVector2D(0.0, c_twoPixels), Mode);
		FVector C23 = SampleTextureLinear(Texture, pixel + FVector2D(c_onePixel, c_twoPixels), Mode);
		FVector C33 = SampleTextureLinear(Texture, pixel + FVector2D(c_twoPixels, c_twoPixels), Mode);
		FVector CP0X = CubicLagrange(C00, C10, C20, C30, frac_.X);
		FVector CP1X = CubicLagrange(C01, C11, C21, C31, frac_.X);
		FVector CP2X = CubicLagrange(C02, C12, C22, C32, frac_.X);
		FVector CP3X = CubicLagrange(C03, C13, C23, C33, frac_.X);

		return CubicLagrange(CP0X, CP1X, CP2X, CP3X, frac_.Y);

	}
```

### line 794
```cpp
inline void ReadColorTextureDataFloat(
		const TVoxelTexture<FColor>& Texture,
		const EVoxelSamplerMode Mode,
		v_flt U,
		v_flt V,
		v_flt& OutR,
		v_flt& OutG,
		v_flt& OutB,
		v_flt& OutA)
	{
		const FLinearColor Color = FLinearColor(BicubicHermiteTextureSample(Texture, FVector2D(U, V), Mode));
		OutR = Color.R;
		OutG = Color.G;
		OutB = Color.B;
		OutA = Color.A;
	}
```

### line 883
```cpp
inline v_flt ReadFloatTextureDataFloat(
		const TVoxelTexture<float>& Texture,
		const EVoxelSamplerMode Mode,
		v_flt U,
		v_flt V)
	{
		return BicubicHermiteTextureSampleLinear(Texture, FVector2D(U, V), Mode).X;
		
	}

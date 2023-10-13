# Mastering Render Targets in Defold: Part 2

In this part we turn our passthrough material into a grayscale material by modifying the fragment shader.

We update `window.fp` with the following grayscale formula from [Wikipedia.](https://en.wikipedia.org/wiki/Grayscale#Luma_coding_in_video_systems)
```glsl
varying mediump vec2 var_texcoord0;

uniform lowp sampler2D texture_sampler;

void main()
{
	vec4 color = texture2D(texture_sampler, var_texcoord0.xy);
	float grayscale = dot(color.rgb, vec3(0.299, 0.587, 0.114));
	gl_FragColor = vec4(grayscale, grayscale, grayscale, color.a);
}
```